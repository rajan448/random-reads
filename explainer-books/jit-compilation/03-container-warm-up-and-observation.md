# 3. Warm-up in containers

> **TL;DR:** Warm realistic request paths under realistic resource limits, then measure startup and sustained performance separately.

## Warm-up needs execution

**Warm-up** is the period when execution develops profiles and optimized code.
**Steady state** means performance becomes reasonably stable for a particular workload.
It does not mean every method reaches C2.

A restaurant improves by practicing actual orders.
Leaving the kitchen open for ten minutes does not practice cooking.
Likewise, sleeping after JVM startup does not exercise your request handlers.

Class loading, static initialization, application caches, and connection pools also affect early requests.
Separate these costs from JIT effects when investigating slow startup.
An ordinary fresh JVM starts with fresh profiling and compiled-code state; an image build does not automatically preserve them.

## CPU limits change the timeline

A **container** packages a process with isolation and resource controls.
On Linux, a **cgroup** controls resource usage for a process group.
A CPU **quota** limits CPU time within a period.
**Throttling** temporarily prevents further execution after that budget is exhausted.
The [Linux cgroup documentation](https://docs.kernel.org/admin-guide/cgroup-v2.html) defines these controls.

Compiler threads, request threads, and **garbage collection**, which reclaims unused heap objects, share available CPU time.
It follows that CPU limits can extend the elapsed time needed to collect profiles and finish compilation.
More compiler threads may increase competition instead of helping.

For illustration, a one-CPU budget supplies roughly one CPU-second of aggregate execution each second.
Adding threads cannot turn that into four CPU-seconds.
Actual timing also depends on scheduling and host contention.

As of November 2026, the JDK 25 HotSpot documentation describes container awareness as enabled by default on Linux.
Check your exact distribution, update, and runtime environment.
`-XX:ActiveProcessorCount=2` overrides the JVM's processor-count view; it does not grant extra CPU quota.
See the [JDK 25 launcher reference](https://docs.oracle.com/en/java/javase/25/docs/specs/man/java.html).

## Observe compilation during a staging run

Replace `app.jar` with your application's executable JAR:

```sh
java -XX:+UnlockDiagnosticVMOptions \
  -XX:+PrintCompilation -XX:+PrintInlining \
  -Xlog:os+container=trace -jar app.jar
```

`PrintCompilation` reports compilation events; `PrintInlining` explains inlining decisions.
The container log shows detected controls on Linux.
Use a bounded diagnostic run because verbose logs add overhead.
These switches are documented in the [launcher reference](https://docs.oracle.com/en/java/javase/25/docs/specs/man/java.html).

In compilation output, levels `3` and `4` identify profiled C1 and C2 compilations in this configuration.
A `%` marks OSR compilation.
“Made not entrant” means new calls cannot enter that compiled version; replacement by newer code is one normal cause.
See the [HotSpot diagnostic walkthrough](https://cr.openjdk.org/~vlivanov/talks/2015_JIT_Overview.pdf).

## A repeatable rollout experiment

1. Start a fresh process with production-like CPU and memory limits.
2. Exercise representative routes, input sizes, branches, and receiver classes using safe test data.
3. Record latency, completed requests per second, compilation activity, allocation, and CPU throttling over time.
4. Repeat fresh starts and compare the curves, rather than one average.
5. Admit traffic gradually once startup dependencies and measured capacity meet your requirements.

**Readiness** means your deployment considers an instance able to serve traffic.
A successful health endpoint alone says little about business-route warm-up.
New replicas can each add another cold period during scaling.
Budget temporary capacity for that transition.

Memory limits need room beyond the Java heap: native code, compiler work, thread stacks, and class information consume memory too.
The **code cache** stores generated native code; increasing heap size cannot fix its capacity.
See Oracle's [native-memory categories](https://docs.oracle.com/en/java/javase/21/troubleshoot/diagnostic-tools.html).

> ⚠️ **Avoid universal tuning recipes.** Capping compilation at C1 with `-XX:TieredStopAtLevel=1` can save compilation work but sacrifice sustained speed. Compare both phases before adopting it.

## Key takeaways

- Idle time cannot substitute for representative execution.
- Resource limits affect warm-up as well as steady-state capacity.
- Readiness and warmed request paths are different measurements.
- Keep cold-start results when benchmarking startup; use warm-up when measuring sustained performance.

For small code benchmarks, explore [OpenJDK JMH](https://github.com/openjdk/jmh), the Java Microbenchmark Harness.
Use separate JVM runs, warm-up iterations, and observable results to reduce misleading measurements.
For services, retain the container-level experiment above.
