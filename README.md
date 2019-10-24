# liburing raw bindings

Rust liburing#24bd087 raw bindings with tests.

# Quick start

```rust
    // init
    let mut ring = unsafe {
        let mut s = mem::MaybeUninit::<io_uring>::uninit();
        let ret = io_uring_queue_init(QUEUE_DEPTH, s.as_mut_ptr(), 0);
        if ret < 0 {
            panic!(
                "io_uring_queue_init: {:?}",
                Error::from_raw_os_error(ret)
            );
        }
        s.assume_init()
    };

    // fill queue
    loop {
        let sqe = unsafe { io_uring_get_sqe(&mut ring) };
        if sqe == std::ptr::null_mut() {
            break;
        }
        unsafe { io_uring_prep_nop(sqe) };
    }

    // submit requests
    let ret = unsafe { io_uring_submit(&mut ring) };
    if ret < 0 {
        panic!("io_uring_submit: {:?}", Error::from_raw_os_error(ret));
    }

    // fetch completions
    let mut cqe: *mut io_uring_cqe = unsafe { std::mem::zeroed() };
    let pending = ret;
    for _ in 0..pending {
        let ret = unsafe { io_uring_wait_cqe(&mut ring, &mut cqe) };
        if ret < 0 {
            panic!(
                "io_uring_wait_cqe: {:?}",
                Error::from_raw_os_error(ret)
            );
        }
        if unsafe { (*cqe).res } < 0 {
            eprintln!("(*cqe).res = {}", unsafe { (*cqe).res });
        }
        unsafe { io_uring_cqe_seen(&mut ring, cqe) };
    }

    unsafe { io_uring_queue_exit(&mut ring) };
```
