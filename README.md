# Tutorial 7

<details>
<summary><b>JMeter Results</b></summary>

## all-student

### Before optimization
![](Screenshot/View%20Results%20in%20Table%20all-student.png)

### After optimization
![](Screenshot/View%20Results%20in%20Table%20all-student-optimized.png)

## all-student-name

### Before optimization
![](Screenshot/View%20Results%20in%20Table%20all-student-name.png)

### After optimization
![](Screenshot/View%20Results%20in%20Table%20all-student-name-optimized.png)

## highest-gpa

### Before optimization
![](Screenshot/View%20Results%20in%20Table%20highest-gpa.png)

### After optimization
![](Screenshot/View%20Results%20in%20Table%20highest-gpa-optimized.png)

</details>

<details>
<summary><b>Profiling Results</b></summary>

## all-student

### Before optimization
![](Screenshot/Profiling%20all-student.png)

### After optimization
![](Screenshot/Profiling%20all-student-optimized.png)

## all-student-name

### Before optimization
![](Screenshot/Profiling%20all-student-name.png)

### After optimization
![](Screenshot/Profiling%20all-student-name-optimized.png)

## highest-gpa

### Before optimization
![](Screenshot/Profiling%20highest-gpa.png)

### After optimization
![](Screenshot/Profiling%20highest-gpa-optimized.png)

</details>

<details>
<summary><b>All Screenshots</b></summary>

## all-student

### View Results Tree
![](Screenshot/View%20Results%20Tree%20all-student.png)

### Summary Report
![](Screenshot/Summary%20Report%20all-student.png)

### Graph Results
![](Screenshot/Graph%20Results%20all-student.png)

### Cli test
![](Screenshot/Cli%20test%20all-student.png)

## all-student-name

### View Results Tree
![](Screenshot/View%20Results%20Tree%20all-student-name.png)

### Summary Report
![](Screenshot/Summary%20Report%20all-student-name.png)

### Graph Results
![](Screenshot/Graph%20Results%20all-student-name.png)

### Cli test
![](Screenshot/Cli%20test%20all-student-name.png)

## highest-gpa

### View Results Tree
![](Screenshot/View%20Results%20Tree%20highest-gpa.png)

### Summary Report
![](Screenshot/Summary%20Report%20highest-gpa.png)

### Graph Results
![](Screenshot/Graph%20Results%20highest-gpa.png)

### Cli test
![](Screenshot/Cli%20test%20highest-gpa.png)

</details>

## Performance Comparison

### Perbandingan Hasil JMeter (Rata-rata Sample Time dalam ms)

| Endpoint | Sebelum | Sesudah | Peningkatan |
|----------|---------|---------|-------------|
| /all-student | 358.930 ms | 1.011 ms | ~99.7% lebih cepat |
| /all-student-name | 7.783 ms | 54 ms | ~99.3% lebih cepat |
| /highest-gpa | 327 ms | 25 ms | ~92.4% lebih cepat |

### Perbandingan Hasil IntelliJ Profiler (CPU Time dalam ms)

| Endpoint | Sebelum | Sesudah | Peningkatan |
|----------|---------|---------|-------------|
| /all-student | 6.687 ms | 450 ms | ~93.3% lebih cepat |
| /all-student-name | 783 ms | 90 ms | ~88.5% lebih cepat |
| /highest-gpa | 270 ms | 90 ms | ~66.7% lebih cepat |

### Kesimpulan

Ketiga endpoint menunjukkan peningkatan performa yang signifikan setelah optimasi, jauh melampaui target peningkatan minimal 20%.

Endpoint `/all-student` menunjukkan peningkatan paling drastis, dari 358.930 ms menjadi hanya 1.011 ms pada JMeter. Hal ini dicapai dengan mengganti N+1 query problem menggunakan satu query `JOIN FETCH`, yang sebelumnya mengirimkan query database terpisah untuk setiap student (20.000 query), kini digantikan dengan satu query yang efisien.

Endpoint `/all-student-name` meningkat signifikan dengan hanya mengambil kolom nama langsung dari database, tanpa memuat seluruh field student, serta mengganti concatenation string yang lambat dengan `String.join`.

Endpoint `/highest-gpa` meningkat dengan mendelegasikan pengurutan dan filtering ke database menggunakan `ORDER BY gpa DESC LIMIT 1`, menggantikan proses memuat 20.000 student ke memori lalu melakukan iterasi di Java.

Secara keseluruhan, optimasi pada level database (query JOIN yang tepat, membatasi kolom yang diambil, mendelegasikan pengurutan ke database) memberikan dampak yang jauh lebih besar dibandingkan optimasi pada level aplikasi saja.

## Reflection

> 1. What is the difference between the approach of performance testing with JMeter and
     profiling with IntelliJ Profiler in the context of optimizing application performance? 

JMeter melakukan pengujian dari sisi luar aplikasi (black-box), mengukur waktu respons dan throughput seperti yang dirasakan oleh pengguna. Sementara itu, IntelliJ Profiler bekerja dari dalam aplikasi (white-box), menunjukkan secara detail method mana yang mengonsumsi CPU time dan memori paling banyak. Keduanya saling melengkapi: JMeter memberi tahu bahwa ada masalah performa, sedangkan IntelliJ Profiler membantu menemukan penyebabnya.

> 2. How does the profiling process help you in identifying and understanding the weak points
     in your application? 

Profiling memungkinkan kita melihat secara langsung method mana yang paling banyak mengonsumsi waktu eksekusi melalui flame graph dan method list. Pada latihan ini, profiling mengungkapkan bahwa `getAllStudentsWithCourses` menjadi bottleneck utama karena melakukan query database secara berulang untuk setiap student, yang dikenal sebagai N+1 query problem.

> 3. Do you think IntelliJ Profiler is effective in assisting you to analyze and identify
     bottlenecks in your application code? 

Ya, IntelliJ Profiler sangat efektif. Fitur flame graph dan method list dengan kolom CPU time memudahkan identifikasi method yang bermasalah secara langsung tanpa harus menebak-nebak. Fitur comparison view juga sangat membantu untuk membuktikan bahwa optimasi yang dilakukan benar-benar memberikan peningkatan performa.

> 4. What are the main challenges you face when conducting performance testing and
     profiling, and how do you overcome these challenges?

Tantangan utama adalah hasil pengukuran yang tidak konsisten akibat JVM warmup pada saat pertama kali aplikasi dijalankan. Selain itu, waktu respons endpoint yang sangat lambat sebelum optimasi membuat proses pengujian memakan waktu lama. Tantangan ini diatasi dengan melakukan pemanasan JVM terlebih dahulu sebelum mengambil pengukuran.

> 5. What are the main benefits you gain from using IntelliJ Profiler for profiling your
     application code?

Manfaat utamanya adalah kemampuan untuk langsung melihat method mana yang menjadi penyebab lambatnya aplikasi beserta waktu eksekusinya secara akurat. IntelliJ Profiler juga terintegrasi langsung dengan IDE sehingga kita bisa langsung melompat ke kode yang bermasalah dan segera melakukan perbaikan tanpa berpindah tool.

> 6. How do you handle situations where the results from profiling with IntelliJ Profiler are not
     entirely consistent with findings from performance testing using JMeter?

Ketidakkonsistenan antara keduanya wajar terjadi karena JMeter mengukur waktu end-to-end termasuk network latency dan overhead HTTP, sedangkan IntelliJ Profiler hanya mengukur waktu eksekusi CPU di dalam aplikasi. Jika terjadi ketidakkonsistenan, analisis dilakukan secara terpisah: profiler digunakan untuk mengoptimasi kode, sementara JMeter digunakan untuk memvalidasi bahwa optimasi tersebut berdampak nyata pada performa dari sisi pengguna.

> 7. What strategies do you implement in optimizing application code after analyzing results
     from performance testing and profiling? How do you ensure the changes you make do
     not affect the application's functionality?

Strategi yang diterapkan adalah mendelegasikan sebanyak mungkin pekerjaan ke database, seperti menggunakan JOIN FETCH untuk menghindari N+1 query, mengambil hanya kolom yang dibutuhkan, dan mendelegasikan pengurutan ke database dengan ORDER BY. Untuk memastikan fungsionalitas tidak terganggu, setiap endpoint diuji kembali setelah optimasi dengan mengakses langsung melalui browser atau Postman untuk memverifikasi bahwa data yang dikembalikan tetap benar dan sesuai.