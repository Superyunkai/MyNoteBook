#### 为什么快
1. 线程局部的日志缓存 —解决加锁问题  **如何理解无锁编程**
nanolog为每个线程都创建了一个线程局部的日志缓存，也就是下面的StagingBuffer，这里使用了C++ 11中的thread_local关键字和gcc特有的关键字_thread
> thread_local T 为每个线程创建一份T的实例，各个线程间的实例具有不同的内存空间，互不干扰，别的线程可以通过取地址符访问到其中的内容
> thread_local 可以与const， static连用
``` C++
// 核心类，运行时的一些日志内存管理，后台日志线程管理
class RuntimeLogger {

private:
    // Storage for staging uncompressed log statements for compression
    static __thread StagingBuffer *stagingBuffer;
    
    // Destroys the __thread StagingBuffer upon its own destruction, which
    // is synchronized with thread death
    static thread_local StagingBufferDestroyer sbc;
    
    //xxx
}
```

光有线程局部的缓存，日志线程就没法访问线程的缓存了，所以还需要一个全局的线程缓存列表，供日志线程访问。因为线程局部缓存 只会创建一次，因此虽然缓存列表是全局需加锁的，日志线程访问它时，加锁的消耗也是可以忽略不记的；

```
// Globally the thread-local stagingBuffers
std::vector<StagingBuffer *> threadBuffers;

inline void
ensureStagingBufferAllocated() {
    if (stagingBuffer == nullptr) {
        std::unique_lock<std::mutex> guard(bufferMutex);
        uint32_t bufferId = nextBufferId++;

        // Unlocked for the expensive StagingBuffer allocation
        guard.unlock();
        stagingBuffer = new StagingBuffer(bufferId);
        guard.lock();

        threadBuffers.push_back(stagingBuffer);
    }
}
```

2. 静态信息和动态信息分离，动态信息延后处理
静态信息处理
下面是Nanolog在工作线程中用于写日志的宏，
``` C++
#define NANO_LOG(severity, format, ...) do { \
    constexpr int numNibbles = NanoLogInternal::getNumNibblesNeeded(format); \
    constexpr int nParams = NanoLogInternal::countFmtParams(format); \
    \
    /*** Very Important*** These must be 'static' so that we can save pointers
     * to these variables and have them persist beyond the invocation.
     * The static logId is used to forever associate this local scope (tied
     * to an expansion of #NANO_LOG) with an id and the paramTypes array is
     * used by the compression function, which is invoked in another thread
     * at a much later time. */ \
    static constexpr std::array<NanoLogInternal::ParamType, nParams> paramTypes = \
                                NanoLogInternal::analyzeFormatString<nParams>(format); \
    static int logId = NanoLogInternal::UNASSIGNED_LOGID; \
    \
    if (NanoLog::severity > NanoLog::getLogLevel()) \
        break; \
    \
    /* Triggers the GNU printf checker by passing it into a no-op function.
     * Trick: This call is surrounded by an if false so that the VA_ARGS don't
     * evaluate for cases like '++i'.*/ \
    if (false) { NanoLogInternal::checkFormat(format, ##__VA_ARGS__); } /*NOLINT(cppcoreguidelines-pro-type-vararg, hicpp-vararg)*/\
    \
    NanoLogInternal::log(logId, __FILE__, __LINE__, NanoLog::severity, format, \
                            numNibbles, paramTypes, ##__VA_ARGS__); \
} while(0)
```
这里获取了几个信息

numNibbles：格式化字符串中，非字符串类型的参数个数，比如%d，%l等参数的个数
nParams： 格式化字符串中，参数的个数，相当于是%的个数
上面这两个都是在编译期即可计算好的，意味着运行时是无消耗的

paramTypes：格式化字符串中，所有参数的参数类型， 即将%..映射为NanoLog内部定义的ParamType
logId：当前日志的格式化字符串的logId，可以理解为一个编号，因为运行时这个信息是不变的，因此我们可以把这些信息存储下来，需要时使用logId索引即可
这里需要注意到，logId是静态的，意味着logId编号会在当前作用域记录下来，即工作线程打印日志的地方，这是必须的，因为运行时需要保证每个打印日志的地方能使用正确的logId获取到正确的格式化字符串信息等
``` C++
template<int NParams, size_t N>
constexpr std::array<ParamType, NParams>
analyzeFormatString(const char (&fmt)[N])
{
    return analyzeFormatStringHelper(fmt, std::make_index_sequence<NParams>{});
}

template<int N, std::size_t... Indices>
constexpr std::array<ParamType, sizeof...(Indices)>
analyzeFormatStringHelper(const char (&fmt)[N], std::index_sequence<Indices...>)
{
    return {{ getParamInfo(fmt, Indices)... }};
}
在获取参数类型信息时，巧妙的使用了std::make_index_sequence自动生成整数序列，自动对参数列表进行遍历，获取信息；并且这些信息也是编译期生成的，因此运行时也是0消耗的

static int logId = NanoLogInternal::UNASSIGNED_LOGID;
NanoLogInternal::log(logId, __FILE__, __LINE__, NanoLog::severity, 
            format, numNibbles, paramTypes, ##__VA_ARGS__);

template<long unsigned int N, int M, typename... Ts>
inline void
log(int &logId,
    const char *filename,
    const int linenum,
    const LogLevel severity,
    const char (&format)[M],
    const int numNibbles,
    const std::array<ParamType, N>& paramTypes,
    Ts... args)
{
    using namespace NanoLogInternal::Log;
    assert(N == static_cast<uint32_t>(sizeof...(Ts)));

    if (logId == UNASSIGNED_LOGID) {
        const ParamType *array = paramTypes.data();
        StaticLogInfo info(&compress<Ts...>,
                        filename,
                        linenum,
                        severity,
                        format,
                        sizeof...(Ts),
                        numNibbles,
                        array);
        printf("LogId : %d", logId);
        RuntimeLogger::registerInvocationSite(info, logId);
    }
     
    //    xxxx
}
```

这是log打印真实调用的函数，这里展示的是静态信息处理部分
可以看到，静态信息（文件名,行号,日志等级,格式化字符串等）并么有做处理，只是注册到了某个地方，相当于就是存到了一个map中，并且这个过程仅会在第一次打印当前日志时处理一次


动态信息处理
``` C++
template<long unsigned int N, int M, typename... Ts>
inline void
log(int &logId,
    const char *filename,
    const int linenum,
    const LogLevel severity,
    const char (&format)[M],
    const int numNibbles,
    const std::array<ParamType, N>& paramTypes,
    Ts... args)
{
//xxx
    uint64_t previousPrecision = -1;
    uint64_t timestamp = PerfUtils::Cycles::rdtsc();
    size_t stringSizes[N + 1] = {}; //HACK: Zero length arrays are not allowed
    size_t allocSize = getArgSizes(paramTypes, previousPrecision,
                            stringSizes, args...) 
                            + sizeof(UncompressedEntry);

    char *writePos = RuntimeLogger::reserveAlloc(allocSize);
    auto originalWritePos = writePos;

    UncompressedEntry *ue = new(writePos) UncompressedEntry();
    writePos += sizeof(UncompressedEntry);

    store_arguments(paramTypes, stringSizes, &writePos, args...);

    ue->fmtId = logId;
    ue->timestamp = timestamp;
    ue->entrySize = downCast<uint32_t>(allocSize);

    assert(allocSize == downCast<uint32_t>((writePos - originalWritePos)));
    NanoLogInternal::RuntimeLogger::finishAlloc(allocSize);
}
```
这个函数就是上面log函数的动态信息处理部分
可以发现，对于动态信息，做了以下处理：

+ 计算日志参数(即传递给printf的参数)需要多大的存储空间（因为要将这些信息写入线程缓存中）
+ 从线程缓存中分配相应大小的内存
+ 将日志参数写入缓存

``` C++
template<int argNum = 0, unsigned long N, 
int M, typename T1, 
typename... Ts>
inline size_t
getArgSizes(const std::array<ParamType, N>& argFmtTypes,
            uint64_t &previousPrecision,
            size_t (&stringSizes)[M],
            T1 head, Ts... rest)
{
    return getArgSize(argFmtTypes[argNum], previousPrecision,
                                                    stringSizes[argNum], head)
           + getArgSizes<argNum + 1>(argFmtTypes, previousPrecision,
                                                    stringSizes, rest...);
}

template<typename T>
inline
typename std::enable_if<!std::is_same<T, const wchar_t*>::value
                        && !std::is_same<T, const char*>::value
                        && !std::is_same<T, wchar_t*>::value
                        && !std::is_same<T, char*>::value
                        && !std::is_same<T, const void*>::value
                        && !std::is_same<T, void*>::value
                        , size_t>::type
getArgSize(const ParamType fmtType,
           uint64_t &previousPrecision,
           size_t &stringSize,
           T arg)
{
    if (fmtType == ParamType::DYNAMIC_PRECISION)
        previousPrecision = as_uint64_t(arg);

    return sizeof(T);
}

##获取日志参数大小时也使用了大量的模板参数推导，以此自动让特定参数类型的日志参数由特定函数处理，简化了代码流程

template<typename T>
inline
typename std::enable_if<!std::is_same<T, const wchar_t*>::value
                        && !std::is_same<T, const char*>::value
                        && !std::is_same<T, wchar_t*>::value
                        && !std::is_same<T, char*>::value
                        , void>::type
store_argument(char **storage,
               T arg,
               ParamType paramType,
               size_t stringSize)
{
    std::memcpy(*storage, &arg, sizeof(T));
    *storage += sizeof(T);

    #ifdef ENABLE_DEBUG_PRINTING
        printf("\tRBasic  [%p]= ", dest);
        std::cout << *dest << "\r\n";
    #endif
}
```
##### 小结
相比普通日志系统，我们会发现，Nanolog在工作线程的运行时，减少了以下部分的消耗：
+ 格式化字符串的传递：NanoLog在运行时并不需要传递格式化字符串
+ 加锁：NanoLog因为是写入线程缓存，并不需要加锁
+ 这两点优化大大降低了NanoLog在运行时工作线程打印日志的延迟
##### 日志线程轻量处理
日志线程有两层主要的循环

第一层循环-遍历线程缓存
``` c++
// Index of the last StagingBuffer checked for uncompressed log messages
size_t lastStagingBufferChecked = 0;

// Manages the state associated with compressing log messages
Log::Encoder encoder(compressingBuffer, NanoLogConfig::OUTPUT_BUFFER_SIZE);

// Keeps a shadow mapping of the log identifiers to static information
// to allow the logging threads to register in parallel with compression
// lookup
std::vector<StaticLogInfo> shadowStaticInfo;

 while (!compressionThreadShouldExit || encoder.getEncodedBytes() > 0
                                        || hasOutstandingOperation)
{
    coreId = sched_getcpu();
    uint64_t bytesConsumedThisIteration = 0;

    std::unique_lock<std::mutex> lock(bufferMutex);
    size_t i = lastStagingBufferChecked;
    
    //xxxx
}
第一层循环自然是从线程缓存列表中那一个缓存出来进行处理

第二层循环-日志压缩
while (!outputBufferFull && !threadBuffers.empty())
{
    uint64_t peekBytes = 0;
    StagingBuffer *sb = threadBuffers[i];
    char *peekPosition = sb->peek(&peekBytes);

    // If there's work, unlock to perform it
    if (peekBytes > 0) {
        uint64_t start = PerfUtils::Cycles::rdtsc();
        lock.unlock();

        // Record metrics on the peek size
        size_t sizeOfDist = Util::arraySize(stagingBufferPeekDist);
        size_t distIndex = (sizeOfDist*peekBytes)/
                                NanoLogConfig::STAGING_BUFFER_SIZE;
        ++(stagingBufferPeekDist[distIndex]);


        // Encode the data in RELEASE_THRESHOLD chunks
        uint32_t remaining = downCast<uint32_t>(peekBytes);
        while (remaining > 0) {
            long bytesToEncode = std::min(
                    NanoLogConfig::RELEASE_THRESHOLD,
                    remaining);
            long bytesRead = encoder.encodeLogMsgs(
                    peekPosition + (peekBytes - remaining),
                    bytesToEncode,
                    sb->getId(),
                    wrapAround,
                    shadowStaticInfo,
                    &logsProcessed);

            wrapAround = false;
            remaining -= downCast<uint32_t>(bytesRead);
            sb->consume(bytesRead);
        }

        lock.lock();
}
```
这里也比较简单，从线程缓存中取出需要处理的数据，进行编码压缩，写入日志线程的一个缓存中；处理完的线程缓存需要通过consume还给线程缓存
**这里有一个论文中提到的细节，可以减少cache miss**
大致意思就是：在从线程缓存中拿数据，和往线程缓存写数据时，因为日志线程和工作线程需要互相知道相互读写指针的位置，那么获取指针位置时，有可能触发cache miss，所以代码中获取指针位置时，使用局部变量缓存了一下，避免cache miss，因为即使指针更新了，使用旧的指针数据也是不影响的
``` C++
char *
RuntimeLogger::StagingBuffer::peek(uint64_t *bytesAvailable) {
    // Save a consistent copy of producerPos
    char *cachedProducerPos = producerPos;

    if (cachedProducerPos < consumerPos) {
        Fence::lfence(); // Prevent reading new producerPos but old endOf...
        *bytesAvailable = endOfRecordedSpace - consumerPos;

        if (*bytesAvailable > 0)
            return consumerPos;

        // Roll over
        consumerPos = storage;
    }

    *bytesAvailable = cachedProducerPos - consumerPos;
    return consumerPos;
}
```
调用异步IO写入文件
``` C++
aioCb.aio_fildes = outputFd;
aioCb.aio_buf = compressingBuffer;
aioCb.aio_nbytes = bytesToWrite;
totalBytesWritten += bytesToWrite;

cyclesAtLastAIOStart = PerfUtils::Cycles::rdtsc();
if (aio_write(&aioCb) == -1)
    fprintf(stderr, "Error at aio_write(): %s\n", strerror(errno));

hasOutstandingOperation = true;

// Swap buffers
encoder.swapBuffer(outputDoubleBuffer,
                   NanoLogConfig::OUTPUT_BUFFER_SIZE);
std::swap(outputDoubleBuffer, compressingBuffer);
outputBufferFull = false;
```
这里做了三件事

调用异步IO接口将当前日志线程缓存中的日志信息写入文件(异步写，并不一定写完了)
标记当前还有操作未完成(有日志在写)
交换日志线程的两个缓存区：因为文件IO是异步的，为了防止文件IO未完成，而日志线程将缓存写坏，这里需要两个缓存进行交换写
在日志线程的下一轮循环中会等待文件IO完成，保证日志信息及时写入磁盘
``` C++
if (hasOutstandingOperation) {
    if (aio_error(&aioCb) == EINPROGRESS) {
        const struct aiocb *const aiocb_list[] = {&aioCb};
        if (outputBufferFull) {
            // If the output buffer is full and we're not done,
            // wait for completion
            cyclesActive += PerfUtils::Cycles::rdtsc() - cyclesAwakeStart;
            int err = aio_suspend(aiocb_list, 1, NULL);
            
aio_suspend(aiocb_list, 1, NULL);即为等待磁盘写入完成
```
#### 总结
NanoLog的一些优化思想是比较值得学习的：
无锁，线程局部缓存去除线程同步间的锁，虽然在访问线程缓存时也需要使用无锁队列的形式，但是这个加锁粒度已经很小了，可以理解为无锁
动静分离，静态信息提前处理，动态信息延后处理，减少运行时的处理逻辑，其实这个和渲染中的各种贴图，比如光照贴图的做法也是类似的
编译期计算：灵活运用编译期计算
cache miss，一般这个是比较难考虑到的，并且是在多线程编程中容易产生性能热点的地方
缺点:
静态信息处理部分使用非C++语言较难实现，若使用工具预先处理由增加了使用的复杂度
日志压缩，一定程度上增加了应用接入kafka，elk等日志工具的成本，在开发期也不太方便