```mermaid
flowchart LR
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
    H3 --> J2
