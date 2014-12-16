# deque - A (mostly) lock-free concurrent work-stealing deque

[![Build Status](https://travis-ci.org/kinghajj/deque.svg?branch=master)](https://travis-ci.org/kinghajj/deque)

This module contains an implementation of the Chase-Lev work stealing deque
described in "Dynamic Circular Work-Stealing Deque". The implementation is
heavily based on the pseudocode found in the paper.

This implementation does not want to have the restriction of a garbage
collector for reclamation of buffers, and instead it uses a shared pool of
buffers. This shared pool is required for correctness in this
implementation.

The only lock-synchronized portions of this deque are the buffer allocation
and deallocation portions. Otherwise all operations are lock-free.

## Example

    use deque::BufferPool;

    let mut pool = BufferPool::new();
    let (mut worker, mut stealer) = pool.deque();

    // Only the worker may push/pop
    worker.push(1i);
    worker.pop();

    // Stealers take data from the other end of the deque
    worker.push(1i);
    stealer.steal();

    // Stealers can be cloned to have many stealers stealing in parallel
    worker.push(1i);
    let mut stealer2 = stealer.clone();
    stealer2.steal();

## History

The `deque` module was originally authored by Alex Crichton on 2013-11-26,
commit a70f9d7324a91058d31c1301c4351932880d57e8 in the Rust git repo.

It was later removed by him as part of the `sync` module rewrite on 2014-11-24
in commit 71d4e77db8ad4b6d821da7e5d5300134ac95974e.

With the introduction of the crates.io package repository, I decided to bring
back the module as its own crate. The code is based on the version in the Rust
repo just prior to the aforementioned commit for the `sync` module rewrite. All
changes so far are only ones required to get the code and tests to compile.
