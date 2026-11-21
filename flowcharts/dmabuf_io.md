```mermaid
flowchart TD
    
    A2(io_read_fixed) -->|1| B1(io_init_rw_fixed)
    A1(io_write_fixed) -->|1|B1

    B1 --> C(__io_import_reg_buf)
    C --> D(io_import_fixed)
    D -->|IO_REGBUF_F_DMABUF| E1(io_import_dmabuf_map)
    D -->|IO_REGBUF_F_KBUF| E2(io_import_kbuf)
    E1 --> F(iov_iter_dmabuf_map)
    
    A2 -->|2| B2(io_read)
    B2 --> C2(__io_read)
    C2 --> D2(io_iter_do_read)
    D2 -->|file->f_op->read_iter|E3{blkdev_read_iter}

    E3 -->|IOCB_DIRECT|F2(kiocb_write_and_wait)
    F2 -->G1{blkdev_direct_IO}
    E3 -->F4(filemap_read)

    G1 -->|nr_pages<256 & \n no metadata|H1{is_sync_kiocb}
    H1 --> |yes| I1(__blkdev_direct_IO_simple)
    H1 --> |no| I2(__blkdev_direct_IO_async)
    
    I1 --> J1(submit_bio_wait)
    I2 --> J2(submit_bio)

    G1 --> H2(__blkdev_direct_IO)
    H2 --> I3(submit_bio\nendio = blkdev_bio_end_io)

    J1 --> H3(bio_await)
    H3 --> J2(submit_bio)

    J2 --> H4(submit_bio_noacct)
    H4 --> I4{submit_bio_noacct_nocheck}
    I4 --> |BD_HAS_SUBMIT_BIO| J3(__submit_bio_noacct)
    I4 --> J4(__submit_bio_noacct_mq)

    J4 --> K1{__submit_bio}
    J3 --> K1
    K1 --> |BD_HAS_SUBMIT_BIO| L1(disk->fops->submit_bio)
    K1 --> L2(blk_mq_submit_bio)
    L2 --> M1(blk_mq_get_new_requests\nblk_mq_run_dispatch_ops)
    M1 --> N1(blk_mq_try_issue_directly)
    N1 --> O1(__blk_mq_issue_directly)
    O1 --> P1(q->mq_ops->queue_rq)

    L1 --> M2(nvme_multipath\nnvme_ns_head_submit_bio)
    M2 --> H4


    P1 --> Q1(nvme_queue_rq)
    Q1 --> R1(nvme_setup_cmd)
    R1 --> S1{nvme_map_data}
    S1 -->|blk_mq_rq_is_dmabuf| T1{nvme_rq_setup_dmabuf}
    T1 -->|sgl| U1(nvme_rq_setup_dmabuf_sgl)
    T1 -->U2(nvme_rq_setup_dmabuf_map)

    S1 -->|segments==1| T2(nvme_pci_setup_data_simple)
    S1 -->|sgl| T3(nvme_pci_setup_data_sgl)
    S1 --> T4(nvme_pci_setup_data_prp)


