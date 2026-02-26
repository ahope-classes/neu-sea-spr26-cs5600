



# 3. Meltdown and Spectre

Handout's memory layout is nice, but...
     ...if the HW isolation is broken, nothing works...
     ...and unfortunately, HW (CPU) today is broken...

   We have Meltdown and Spectre (2018).
     see: https://meltdownattack.com/

"""
Q: Am I affected by the vulnerability?
A: Most certainly, yes.

Q: Can I detect if someone has exploited Meltdown or Spectre against me?
A: Probably not. The exploitation does not leave any traces in traditional log files.

Q: What can be leaked?
A: If your system is affected, our proof-of-concept exploit can read the memory
content of your computer. This may include passwords and sensitive data stored
on the system.

Q: Has Meltdown or Spectre been abused in the wild?
A: We don't know.
"""

   -- backgrounds

     -- side channel attack

        We have caches all over the places to accelerate memory accesses.

        Cache side-channel attacks exploit timing differences that are
        introduced by the caches.

        An attacker frequently flushes a targeted memory location.

        By measuring the time of reloading the data, another process (the
        attacker) can determine whether data was loaded into the cache.

     -- speculative execution

        Motivation:

        given a piece of code:

        if (read_a_bool_from_memory) {
          foo()
        }

        reading from memory can be slow (hundreds of cycles).
        Before knowing the result, in principle, CPU could do nothing.

        In fact, CPU will predict the branch and might speculatively run foo():
          if bool is false, CPU discards all the results of foo() => as if nothing happens
          if bool is true, hooray, CPU save a lot of time!

        -- Speculative execution on modern CPUs can run several hundred
        instructions ahead.

   -- spectre: speculation + time channel
      (simplied and pseudocode)

      -- first, run code:

         if (x < array1_size) {
            y = array2[array1[x] * 4096];
         }

        ** where x is way larger than array1_size, which ends up on some secret.
           (meaning the address of (array1 + x) points to some secret)
        ** we have all pages in array2 uncached

      -- then test which page has been touched:

        for (int i=0; i<256; i++) {
            test how long to read array2[i << 12]
        }

        if ith round is faster than others,
          then we know the secrete is "i"

    [skipped most of the pieces]
    -- meltdown: out-of-order execution + time channel
      (simplied and pseudocode)

       --out-of-order execution
         CPU doesn't have to run code line by line.
         It might be running them out-of-order to accelerate the execution.

       --first, run code:

        byte = read_one_byte_from_kernel() // will throw exception
        // the line below should have been never reached 
        int x = array[byte << 12]

       --then, run:

        for (int i=0; i<256; i++) { // 256 = 2^8 (8bits = 1byte)
            test how long to read array[i << 12]
        }

    -- mitigation:
      [will talk about it next time]


# 1. Last time

  -- TLB
    -- Virtual memory overview
      [show slides]

  -- where does OS live?

  -- meltdown and spectre

  --post-meltdown kernel (mitigation: KAISER)

    notes from Aurojit Panda about kernel-being-mapped-into-each-process:

      [AP: This answer is complicated
      (https://www.usenix.org/system/files/login/articles/login_winter18_03_gruss.pdf),
      but see below for attempt to explain.

    * In the post-meltdown KAISER/KPTI/KVA/XNU Double Map (all names for
      similar mitigations) each process has two (logical) page tables:

        - One, the user mode page table, for use when the process is
        executing usermode code, unmaps most (but not all) of the kernel,
        this includes some of the kernel stack, and a few other things. The
        aim of all mitigations has been to minimize the number of kernel
        pages in the user mode page table, but different tradeoffs are
        selected for how much this is minimized.

        - The second, the kernel mode page table, has exactly the same
        layout as what [is mentioned above], i.e., the kernel is in all address
        spaces.

    On entry to kernel, the OS tries as rapidly as possible to switch from
    user mode page table to kernel mode one.

    It switches back before return. Having a kernel mode page table per
    process is in part to minimize how much one needs to change in the
    kernel, so maybe one can argue that this is not the "best" possible
    solution, but it is pretty good.]



