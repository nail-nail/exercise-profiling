# Tutorial 7

<details>
<summary><b>JMeter Results</b></summary>

## all-student

### Before optimization
![](img/all-students-before-opt.png)

### After optimization
![](img/all-s-table-optimized.png)

## all-student-name

### Before optimization
![](img/all-sname-table-noopt.png)

### After optimization
![](img/all-sname-table-opt.png)

## highest-gpa

### Before optimization
![](img/all-gpa-table-noopt.png)

### After optimization
![](img/gpa-table-optimized.png)

</details>

<details>
<summary><b>Profiling Results</b></summary>

## all-student

### Before optimization
![](img/profile-all-s-noopt.png)

### After optimization
![](img/prof-all-s-opt.png)

## all-student-name

### Before optimization
![](img/prof-allsname-noopt.png)

### After optimization
![](img/prof-all-sname-opt.png)

## highest-gpa

### Before optimization
![](img/prof-gpa-noopt.png)

### After optimization
![](img/profile-gpa-opt.png)

</details>

<details>
<summary><b>All Screenshots</b></summary>

## all-student

### View Results Tree
![](img/all-s-tree-before-opt.png)

### Summary Report
![](img/all-s-no-opt-summary.png)

### Graph Results
![](img/all-s-noop-graph-result.png)

### Cli test
![](img/all-s-cli-noopt.png)

### View Results Tree (optimized)
![](img/all-s-tree-opt.png)

### Summary Report (optimized)
![](img/all-s-summary-opt.png)

### Graph Results (optimized)
![](img/all-s-graph-optimized.png)

### Cli test (optimized)
![](img/all-s-cli.png)

## all-student-name

### View Results Tree
![](img/all-sname-noopt.png)

### Summary Report
![](img/all-sname-summary-noopt.png)

### Graph Results
![](img/all-sname-graph-noopt.png)

### Cli test
![](img/all-sname-cli-noopt.png)

### View Results Tree (optimized)
![](img/all-sname-tree-opt.png)

### Summary Report (optimized)
![](img/all-sname-summary-optimized.png)

### Graph Results (optimized)
![](img/all-sname-graph-opt.png)

### Cli test (optimized)
![](img/cli-stname-opt.png)

## highest-gpa

### View Results Tree
![](img/all-gpa-tree-noopt.png)

### Summary Report
![](img/all-gpa-summary-noopt.png)

### Graph Results
![](img/gpa-graph-noopt.png)

### Cli test
![](img/highestgpanoopt-cli.png)

### View Results Tree (optimized)
![](img/gpa-tree-optimized.png)

### Summary Report (optimized)
![](img/gpa-summary-optimized.png)

### Graph Results (optimized)
![](img/gpa-graph-opt.png)

### Cli test (optimized)
![](img/cli-highestgpa-opt.png)

</details>

## Performance Comparison

### JMeter

| Endpoint | Before | After | Improvement |
|----------|---------|---------|-------------|
| `/all-student` | 79.801,5 ms | 1.730,6 ms | ~97,8% faster |
| `/all-student-name` | 6.195,5 ms | 43,1 ms | ~99,3% faster |
| `/highest-gpa` | 100,6 ms | 26,9 ms | ~73,2% faster |

### Profiler 
| Endpoint | Before | After | Improvement |
|----------|---------|---------|-------------|
| `/all-student` | 11.325 ms | 768 ms | ~93,2% faster |
| `/all-student-name` | 1.308 ms | 255 ms | ~80,5% faster |
| `/highest-gpa` | 504 ms | 187 ms | ~62,9% faster |

### Conclusion

This section changes how I approach application performance, moving from just writing functional code to actively diagnosing and optimizing it. Previously, I might have relied entirely on external testing or assumptions, but by using IntelliJ Profiler, I learned how to look inside the JVM to see exactly what the application is doing. The profiler revealed exactly which methods such as `getAllStudentsWithCourses`, `joinStudentNames`, and `findStudentWithHighestGpa` that were consuming the most CPU time. I learned that while external load testing tools like JMeter can tell you *that* an endpoint is slow, the profiler is essential for pointing out exactly *where* and *why* the bottleneck exists inside the code.

The most important idea I learned here is that poorly optimized data fetching and processing can absolutely cripple an application, even if the underlying logic is technically correct. For instance, the original implementation of the endpoints resulted in massive delays, with JMeter recording sample times taking nearly 80,000ms under load and dominating the CPU flame graph. By refactoring the code to handle data more efficiently (resolving inefficient loops and redundant database calls) the CPU time for `getAllStudentsWithCourses` dropped drastically from 11,450ms down to just 768ms. Similar optimizations drastically reduced the footprint of the other methods as well.

The bigger lesson is that true scalability requires a combination of macro-level load testing and micro-level code profiling. It is easy to write code that works perfectly for a single request but fails under load due to hidden inefficiencies. By validating the refactored code with JMeter, I could see the tangible impact of these code-level changes: request sample times dropped from thousands of milliseconds down to the 20ms to 120ms range. That makes the application significantly more practical and demonstrates that writing efficient, profiled code is just as critical to server performance as the architectural design itself.

## Reflection

> 1. What is the difference between the approach of performance testing with JMeter and
     profiling with IntelliJ Profiler in the context of optimizing application performance?

JMeter operates as a black-box tool, simulating concurrent users hitting endpoints from the outside and measuring response time and throughput as the end user would experience them; it tells you *that* something is slow. IntelliJ Profiler works from inside the JVM, recording exactly how much CPU time and memory each method consumes during execution, telling you *where* and *why* the slowness originates. The two are complementary: JMeter surfaces the symptom under realistic load, while the profiler pinpoints the root cause in the code.

> 2. How does the profiling process help you in identifying and understanding the weak points
     in your application?

The profiler's flame graph and method list make it immediately obvious which methods dominate CPU time, removing any need to guess. In this exercise, it directly exposed `getAllStudentsWithCourses` as the primary bottleneck by showing it consumed the overwhelming majority of CPU cycles caused by an N+1 query pattern that issued a separate database call for every student. Without the profiler, that kind of hidden inefficiency would be nearly impossible to locate just by reading the code or looking at JMeter numbers alone.

> 3. Do you think IntelliJ Profiler is effective in assisting you to analyze and identify
     bottlenecks in your application code?

Yes, it is very effective. The combination of the flame graph for a visual overview, the method list for precise CPU time figures, and the comparison view for validating improvements covers everything needed to diagnose and confirm a fix. Being integrated directly into IntelliJ also means I can jump from a profiling result straight to the problematic source line without switching tools, which keeps the feedback loop tight and fast.

> 4. What are the main challenges you face when conducting performance testing and
     profiling, and how do you overcome these challenges?

The main challenge is measurement inconsistency caused by JVM warmup. The JIT compiler has not yet optimized hot paths on the first few runs, so initial numbers are misleadingly slow and not representative of steady-state performance. I overcame this by starting the application, hitting each endpoint a few times to warm the JVM, then restarting and taking the actual measurement from a warmed-up state. Slow pre-optimization response times also made the test cycle tedious, which was simply a cost of working with unoptimized code before the fixes were in.

> 5. What are the main benefits you gain from using IntelliJ Profiler for profiling your
     application code?

The biggest benefit is precision. Rather than broad approximations, I get exact CPU time and call counts per method, making it trivial to rank bottlenecks by actual impact rather than intuition. The comparison view is particularly valuable because it removes subjectivity from evaluating a fix: you can place the before and after sessions side by side and see the numeric reduction directly. The IDE integration also means no context-switching, so the cycle of profile → identify → fix → re-profile stays fast.

> 6. How do you handle situations where the results from profiling with IntelliJ Profiler are not
     entirely consistent with findings from performance testing using JMeter?

Inconsistency between the two is expected rather than alarming, because they measure fundamentally different things. JMeter captures full end-to-end latency including network overhead, HTTP processing, and serialization, while the profiler only accounts for in-process CPU time. When the numbers diverge, I treat them as separate signals: use the profiler to optimize the internal code, then use JMeter to confirm that the internal improvement actually translates to a meaningful reduction in user-facing response time. If JMeter still shows no improvement after the profiler shows a gain, the remaining bottleneck is likely outside the application code itself.

> 7. What strategies do you implement in optimizing application code after analyzing results
     from performance testing and profiling? How do you ensure the changes you make do
     not affect the application's functionality?

The core strategy I applied was to push as much work as possible to the database rather than doing it in Java loops. Replacing the N+1 query pattern with a single `JOIN FETCH`, delegating sorting to the database with `ORDER BY gpa DESC LIMIT 1`, and fetching only the required column with a projection query instead of loading full entities. To ensure correctness was not broken, I manually re-tested each endpoint after each change by hitting it through a browser and Postman to verify the response data was identical to before the refactor, then re-ran JMeter to confirm the performance gain was real and consistent.


