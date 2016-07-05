mm_init
========================================

Arguments
----------------------------------------

path: kernel/fork.c
```
static struct mm_struct *mm_init(struct mm_struct *mm, struct task_struct *p)
{
    mm->mmap = NULL;
    mm->mm_rb = RB_ROOT;
    mm->vmacache_seqnum = 0;
    atomic_set(&mm->mm_users, 1);
    atomic_set(&mm->mm_count, 1);
    init_rwsem(&mm->mmap_sem);
    INIT_LIST_HEAD(&mm->mmlist);
    mm->core_state = NULL;
    atomic_long_set(&mm->nr_ptes, 0);
    mm_nr_pmds_init(mm);
    mm->map_count = 0;
    mm->locked_vm = 0;
    mm->pinned_vm = 0;
    memset(&mm->rss_stat, 0, sizeof(mm->rss_stat));
    spin_lock_init(&mm->page_table_lock);
    mm_init_cpumask(mm);
    mm_init_aio(mm);
    mm_init_owner(mm, p);
    mmu_notifier_mm_init(mm);
    clear_tlb_flush_pending(mm);
#if defined(CONFIG_TRANSPARENT_HUGEPAGE) && !USE_SPLIT_PMD_PTLOCKS
    mm->pmd_huge_pte = NULL;
#endif

    if (current->mm) {
        mm->flags = current->mm->flags & MMF_INIT_MASK;
        mm->def_flags = current->mm->def_flags & VM_INIT_DEF_MASK;
    } else {
        mm->flags = default_dump_filter;
        mm->def_flags = 0;
    }
```

mm_alloc_pgd
----------------------------------------

```
    if (mm_alloc_pgd(mm))
        goto fail_nopgd;
```

https://github.com/novelinux/linux-4.x.y/blob/master/kernel/fork.c/mm_alloc_pgd.md

init_new_context
----------------------------------------

```
    if (init_new_context(p, mm))
        goto fail_nocontext;

    return mm;

fail_nocontext:
    mm_free_pgd(mm);
fail_nopgd:
    free_mm(mm);
    return NULL;
}
```