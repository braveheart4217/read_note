	### LevelDB 源码分析###

1. 目录层次
-------

- db	  //db逻辑相关

- helpers //内存db接口

- include //Interface

- port    //操作系统相关的移植接口

- table   //表存储结构

- util    //公用组件


结构说明：

Hashtable 结构
---------- ------------ ---------- ---------------------- -----

   node0  |   node1  |  node2  |  node3  |  node4  |  node5  |

------|---- -----|----------------- ---------------------- ----


	// Cache 节点元素结构体
	struct LRUHandle {
      	void* value;           //value值指针
      	void (*deleter)(const Slice&, void* value);
      	LRUHandle* next_hash;  //指向链表的下一个节点（使用链表的方法解决冲突）
     	LRUHandle* next;       //下一个节点
     	LRUHandle* prev;       //前一个节点
      	size_t charge;         // TODO(opt): Only allow uint32_t?
     	size_t key_length;     //Key 的长度
      	uint32_t refs;         //引用次数
      	uint32_t hash;         // Hash of key(); used for fast sharding and comparisons
      	char key_data[1];      // Beginning of key，相当于占位符，后面会把真正的Key数据复制过来

    };

    Class HashTable{
    private:
      // The table consists of an array of buckets where each bucket is
      // a linked list of cache entries that hash into the bucket.
      uint32_t length_;  //Hash 数组的大小，数组每个元素下面跟着链表
      uint32_t elems_;   //当此值大于length_时会做resize
      LRUHandle** list_; //指向hash数组头
    }

    class LRUCache {
    
     private:
      // Initialized before use.
      size_t capacity_;

      // mutex_ protects the following state.
      port::Mutex mutex_;
      size_t usage_;
    
      // Dummy head of LRU list.
      // lru.prev is newest entry, lru.next is oldest entry.
      LRUHandle lru_;
    
      HandleTable table_;
    };

LRUCache 是对 HashTable 的一个缓存
usage,charge作用？ 

// uasge ： 当前使用大小.+=Insert参数的charge.

// charge： usage_的最大值.


LRU_Cache 与HashTable的关系？LRUCache缓存了所有HashTable里面的东西？为什么要对HashTable做完全缓存？这样做的意义？
[Level源码分析](http://blog.csdn.net/sparkliang/article/details/8567602)

note：HashTable是用来快速查找一个LRUHandle是否在cache中，所以HashTable肯定是相当于Cache 的一份Copy

    while (usage_ > capacity_ && lru_.next != &lru_) {
    	LRUHandle* old = lru_.next;
    	LRU_Remove(old);                       //为什么LRU队列与HashTable里面都要移除？
		table_.Remove(old->key(), old->hash);
    	Unref(old);
     }

###########################
    static const int kNumShardBits = 4;
    static const int kNumShards = 1 << kNumShardBits;

ShardedLRUCache成员变量

    class ShardedLRUCache : public Cache {  //相当于MySQL的分库，通过Key的Hash值得前五位来分配此Key应该存放在哪个Shard里面
     private:
      LRUCache shard_[kNumShards];  //这里是所有shard的数组
      port::Mutex id_mutex_;        //锁
      uint64_t last_id_;
    }

################

    class FileState{
    	private:
			port::Mutex refs_mutex_; //内部锁
			int refs_;  // Protected by refs_mutex_;

			//在写入log时会先分配一个 log block块(8K),然后将指针其加入blocks_，以便后期写入磁盘
			std::vector<char*> blocks_;
			//size 指示 log 文件的总偏移量，也就是已经写入日志文件数据的所以字节数，也就是日志文件的大小
			uint64_t size_;
			enum { kBlockSize = 8 * 1024 };
    }
###
    class SequentialFileImpl : public SequentialFile{
    	private:
			//日志文件的状态
    		FileState* file_;
			//设置日志文件读取的位置，也就是说日志文件总是从pos_位置开始读取的，也是每次读取日志之前都要
			// Status s = file_->Read(pos_, n, result, scratch);
    		size_t pos_;
    };

关于log:reader 中readRecord的一个问题？
为什么每次申请空间是8K，为什么直接申请32K的大小
SkipList的构造过程？

	// sstable block format代码,block构造类
    class block_builder {
    private:
    	const Options*options_;
    	std::string   buffer_;  // Destination buffer sstable存储数据的最终位置
    	std::vector<uint32_t> restarts_;// Restart points restarts_[i] 里面装的是第i个 restartPoint 在 data block的偏移量
    	int   counter_; // Number of entries emitted since restart
    	bool  finished_;// Has Finish() been called?
    	std::string   last_key_; //记录最后添加的key 
    }
####

    //block 读取类
    class Block {
     private:
      uint32_t NumRestarts() const;
    
      const char* data_;  // block数据指针
      size_t size_;   // block数据大小
      uint32_t restart_offset_; // the Offset of restart array in data_ 
      bool owned_;  // Block owns data_[] block 数据是否是在堆上分配，这决定了在析构的时候要不要释放内存
    
      class Iter;
    };

#####
	// 迭代器类
    class Iterator{
        private:
            struct Cleanup {
       			CleanupFunction function;
       		 	void* arg1;
        		void* arg2;
    			Cleanup* next;
      		};
            Cleanup cleanup_;
    }
	// block Iterator类
    class Block::Iter : public Iterator {
     private:
        const Comparator* const comparator_;
        const char* const data_;      // underlying block contents  当前block指针
        uint32_t const restarts_;     // Offset of restart array (list of fixed32)  restart point array的偏移量
        uint32_t const num_restarts_; // Number of uint32_t entries in restart array
    
        // current_ is offset in data_ of current entry.  >= restarts_ if !Valid
        uint32_t current_;         // 当前记录的偏移量
        uint32_t restart_index_;   // Index of restart block in which current_ falls
								   // 指示当前记录在第 restart_index_ 个restart point内
        std::string key_;          // 当前 Iterator 指示记录的 key
        Slice value_;              // 当前 Iterator 指示记录的 value的指针
        Status status_;
    }
#####

	// blockHandle 实际上就是一下位置索引
	class BlockHandle {

	 // Maximum encoding length of a BlockHandle
	 enum { kMaxEncodedLength = 10 + 10 };
      private:
        uint64_t offset_;  // the offset of the block in the file
        uint64_t size_;  //the size of the stored block
	};
###
	//footer class
    class Footer {
      enum {kEncodedLength = 2*BlockHandle::kMaxEncodedLength + 8};
     private:
      BlockHandle metaindex_handle_;
      BlockHandle index_handle_;
    };

####
	static const uint64_t kTableMagicNumber = 0xdb4775248b80fb57ull;
	// 1-byte type + 32-bit crc
	static const size_t kBlockTrailerSize = 5;
####

	//sstable 构造类
    struct TableBuilder::Rep {
        Options options;  // data block 的选项
        Options index_block_options; // index block的选项
        WritableFile* file;//sstalbe 文件指针
        uint64_t offset;  // 要写入的data block在sstable 中的偏移
        Status status;    // 当前状态-初始状态为ok
        BlockBuilder data_block;   // 当前操作的 data block 
        BlockBuilder index_block;  // sstable 的 索引区
        std::string last_key;      // 当前data block最后插入的kv 的key
        int64_t num_entries;       // 当前data block的个数
        bool closed;  // Either Finish() or Abandon() has been called.
        FilterBlockBuilder* filter_block; //根据filter快速定位key是否在block中
    
        // Invariant: r->pending_index_entry is true only if data_block is empty.
        bool pending_index_entry;
        BlockHandle pending_handle; // 添加到index block的data block信息
        std::string compressed_output;//压缩后的data block，临时存储，写入后即被清空

    }

