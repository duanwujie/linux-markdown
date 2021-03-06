struct mm_struct
========================================

进程的虚拟地址空间被分成一个个的区域，用户进程在虚拟地址空间的
一个区域使用vm_area_struct来表示，所有的区域使用两种数据结构来
共同表示，一种是单链表，另外一种是红黑树. 具体表示如下所示：

mmap
----------------------------------------

path: include/linux/mm_types.h
```
struct mm_struct {
    struct vm_area_struct *mmap;      /* list of VMAs */
```

单链表的头节点. 用户虚拟地址空间中的每一个区域由开始地址和结束
地址描述.现存的区域按起始地址以递增的顺序被归入链表中.

每个区域表示为vm_area_struct的一个实例，如下所示:

https://github.com/novelinux/linux-4.x.y/tree/master/include/linux/mm_types.h/struct_vm_area_struct.md

mm_rb
----------------------------------------

```
    struct rb_root mm_rb;
```

红黑树的根节点. 扫描链表找到与特定地址关联的区域，在有大量区域
的时候是非常低效的操作，因此vm_area_struct的各个实例还是通过
红黑树管理，可以显著加快扫描速度.

vmacache_seqnum
----------------------------------------

```
    u32 vmacache_seqnum;              /* per-thread vmacache */
```

get_unmapped_area
----------------------------------------

```
#ifdef CONFIG_MMU
    unsigned long (*get_unmapped_area) (struct file *filp,
                unsigned long addr, unsigned long len,
                unsigned long pgoff, unsigned long flags);
#endif
```

mmap_base
----------------------------------------

```
    unsigned long mmap_base;        /* base of mmap area */
```

表示虚拟地址空间中用于内存映射的起始地址,可调用get_unmapped_area
在mmap区域中为新映射找到适当的位置. 该值通常设置为TASK_UNMAPPED_BASE,
每个体系结构都需要定义，几乎所有情况其值都是TASK_SIZE / 3.

**注意**: 如果使用内核的默认配置，则mmap区域的起始点不是随机的.

mmap_legacy_base
----------------------------------------

```
    unsigned long mmap_legacy_base; /* base of mmap area in bottom-up allocations */
```

task_size
----------------------------------------

```
    unsigned long task_size;        /* size of task vm space */
```

存储了对应进程的地址空间长度，该值通常是TASK_SIZE.

highest_vm_end
----------------------------------------

```
    unsigned long highest_vm_end;  /* highest vma end address */
```

pgd
----------------------------------------

```
    pgd_t * pgd;
```

mm_users
----------------------------------------

```
    atomic_t mm_users;     /* How many users with user space? */
    atomic_t mm_count;    /* How many references to "struct mm_struct" (users count as 1) */
    atomic_long_t nr_ptes;        /* Page table pages */
    int map_count;                /* number of VMAs */

    spinlock_t page_table_lock;     /* Protects page tables and some counters */
    struct rw_semaphore mmap_sem;

    struct list_head mmlist;        /* List of maybe swapped mm's.    These are globally strung
                         * together off init_mm.mmlist, and are protected
                         * by mmlist_lock
                         */


    unsigned long hiwater_rss;   /* High-watermark of RSS usage */
    unsigned long hiwater_vm;    /* High-water virtual memory usage */
```

total_vm
----------------------------------------

```
    unsigned long total_vm;     /* Total pages mapped */
```

进程虚拟地址空间的页数.

locked_vm
----------------------------------------

```
    unsigned long locked_vm;    /* Pages that have PG_mlocked set */
    unsigned long pinned_vm;    /* Refcount permanently increased */
    unsigned long shared_vm;    /* Shared pages (files) */
    unsigned long exec_vm;      /* VM_EXEC & ~VM_WRITE */
```

stack_vm
----------------------------------------

```
    unsigned long stack_vm;     /* VM_GROWSUP/DOWN */
```

用户态堆栈中的页数.

def_flags
----------------------------------------

```
    unsigned long def_flags;
```

start_code, end_code, start_data, end_data
----------------------------------------

```
    unsigned long start_code, end_code, start_data, end_data;
```

可执行代码占用的虚拟地址空间区域其开始和结束通过start_code和
end_code标记. 类似地，start_data和end_data标记了包含已经初始化
数据的区域.

**注意**: 在ELF二进制文件映射到地址空间中之后，这些区域长度不再改变.

start_brk, brk, start_stack
----------------------------------------

```
    unsigned long start_brk, brk, start_stack;
```

堆的起始地址保存在start_brk, brk表示堆区域当前的结束地址。
尽管堆的起始地址在进程生命周期是不变的，但堆的长度会发生改变，
因此brk的值也会改变.

栈的起始地址保存在start_stack中，通常，栈起始于STACK_TOP,如果
设置了PF_RANDOMIZE,则起始点会减少一个小的随机量.每个体系结构都
必须定义STACK_TOP,大多数都设置为TASK_SIZE即用户地址空间中最高的
可用地址。进程的参数列表和环境变量都是栈的初始数据.

arg_start, arg_end, env_start, env_end
----------------------------------------

```
    unsigned long arg_start, arg_end, env_start, env_end;
```

参数列表和环境变量的位置分别由arg_start,arg_end和env_start,
env_end描述.两个区域都位于栈中的最高区域.

saved_auxv
----------------------------------------

```
    unsigned long saved_auxv[AT_VECTOR_SIZE]; /* for /proc/PID/auxv */

    /*
     * Special counters, in some configurations protected by the
     * page_table_lock, in other configurations by being atomic.
     */
    struct mm_rss_stat rss_stat;

    struct linux_binfmt *binfmt;

    cpumask_var_t cpu_vm_mask_var;

    /* Architecture-specific MM context */
    mm_context_t context;

    unsigned long flags; /* Must use atomic bitops to access the bits */

    struct core_state *core_state; /* coredumping support */
#ifdef CONFIG_AIO
    spinlock_t            ioctx_lock;
    struct kioctx_table __rcu    *ioctx_table;
#endif
#ifdef CONFIG_MEMCG
    /*
     * "owner" points to a task that is regarded as the canonical
     * user/owner of this mm. All of the following must be true in
     * order for it to be changed:
     *
     * current == mm->owner
     * current->mm != mm
     * new_owner->mm == mm
     * new_owner->alloc_lock is held
     */
    struct task_struct __rcu *owner;
#endif
```

exe_file
----------------------------------------

```
    /* store ref to file /proc/<pid>/exe symlink points to */
    struct file *exe_file;
```

mmu_notifier_mm
----------------------------------------

```
#ifdef CONFIG_MMU_NOTIFIER
    struct mmu_notifier_mm *mmu_notifier_mm;
#endif
#if defined(CONFIG_TRANSPARENT_HUGEPAGE) && !USE_SPLIT_PMD_PTLOCKS
    pgtable_t pmd_huge_pte; /* protected by page_table_lock */
#endif
#ifdef CONFIG_CPUMASK_OFFSTACK
    struct cpumask cpumask_allocation;
#endif
#ifdef CONFIG_NUMA_BALANCING
    /*
     * numa_next_scan is the next time that the PTEs will be marked
     * pte_numa. NUMA hinting faults will gather statistics and migrate
     * pages to new nodes if necessary.
     */
    unsigned long numa_next_scan;

    /* Restart point for scanning and setting pte_numa */
    unsigned long numa_scan_offset;

    /* numa_scan_seq prevents two threads setting pte_numa */
    int numa_scan_seq;
#endif
#if defined(CONFIG_NUMA_BALANCING) || defined(CONFIG_COMPACTION)
    /*
     * An operation with batched TLB flushing is going on. Anything that
     * can move process memory needs to flush the TLB when moving a
     * PROT_NONE or PROT_NUMA mapped page.
     */
    bool tlb_flush_pending;
#endif
    struct uprobes_state uprobes_state;
#ifdef CONFIG_X86_INTEL_MPX
    /* address of the bounds directory */
    void __user *bd_addr;
#endif
};
```
