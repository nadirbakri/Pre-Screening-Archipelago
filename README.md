
# Developer Screening Questions

## Database Solutions

### Level 1

```sql
SELECT COUNT(*) FROM Customers WHERE Country = 'Germany';
```

### Level 2

```sql
SELECT COUNT(CustomerID) as 'COUNT(CustomerID)', Country
FROM Customers 
GROUP BY Country 
HAVING COUNT(CustomerID) >= 5
ORDER BY COUNT(CustomerID) DESC;
```

### Level 3

```sql
SELECT TOP 9
    Customers.CustomerName,
    COUNT(Orders.OrderID) as OrderCount,
    FORMAT(MIN(Orders.OrderDate), 'yyyy-MM-dd') as FirstOrder,
    FORMAT(MAX(Orders.OrderDate), 'yyyy-MM-dd') as LastOrder
FROM Customers
INNER JOIN Orders ON Customers.CustomerID = Orders.CustomerID
GROUP BY Customers.CustomerID, Customers.CustomerName
HAVING COUNT(Orders.OrderID) >= 5
ORDER BY COUNT(Orders.OrderID) DESC;
```

## JavaScript/TypeScript Questions

### Level 1

```javascript
function titleCase(str) {
    return str.toLowerCase().split(' ').map(word => {
        return word.charAt(0).toUpperCase() + word.slice(1);
    }).join(' ');
}
```

```javascript
function wordFrequency(str) {
    const words = str.toLowerCase().replace(/[^\w\s]/g, '').split(/\s+/);
    const frequency = {};
    words.forEach(word => {
        if (word) {
            frequency[word] = (frequency[word] || 0) + 1;
        }
    });
    
    return frequency;
}
```

### Level 2

```javascript
function delay(ms) {
    return new Promise(resolve => {
        setTimeout(resolve, ms);
    });
}

delay(3000).then(() => alert('runs after 3 seconds'));
```

### Level 2.5

```javascript
async function fetchData(url) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (!url) {
                reject("URL is required");
            } else {
                resolve(`Data from ${url}`);
            }
        }, 1000);
    });
}

async function processData(data) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (!data) {
                reject("Data is required");
            } else {
                resolve(data.toUpperCase());
            }
        }, 1000);
    });
}

fetchData("https://example.com")
    .then(async (data) => {
        console.log("Fetch Success:", data);
        const processedData = await processData(data);
        console.log("Processed Data:", processedData);
    })
    .catch(err => console.error("Error:", err));
```

### Level 3

![Demo Application](https://i.postimg.cc/1tzsTq3Y/image.png) [GitHub Repository](https://github.com/nadirbakri/real-time-chat)

## Vue.js

### Vue.js Reactivity and Common Issues

Vue.js mengimplementasikan sistem reactivity menggunakan **Proxy objects** pada Vue 3 atau **Object.defineProperty** pada Vue 2 untuk memantau perubahan data dan memicu pembaruan komponen.

**Cara kerja:**

-   Vue membungkus data reaktif dengan proxy yang mendeteksi akses properti
-   Saat komponen di-render, Vue mencatat properti reaktif yang digunakan
-   Ketika terjadi perubahan pada properti tersebut, Vue akan memicu re-render pada komponen terkait
-   Sistem ini membangun grafik ketergantungan antara data reaktif dengan computed properties dan komponen

**Masalah pelacakan yang sering terjadi:**

-   **Pengaturan index array di Vue 2:** `arr[0] = newValue` tidak mengaktifkan reactivity (gunakan `Vue.set()` atau metode array)
-   **Penambahan properti baru pada objek:** `obj.newProp = value` tidak reaktif di Vue 2 (gunakan `Vue.set()` atau `this.$set()`)
-   **Destructuring objek reaktif:** `const { count } = reactive({ count: 0 })` menghilangkan reactivity
-   **Penugasan langsung pada refs:** `const count = ref(0); count = ref(1)` merusak reactivity (seharusnya `count.value = 1`)
-   **Masalah timing async:** Mengubah data reaktif dalam setTimeout/Promise callbacks dapat menimbulkan perilaku tidak terduga
-   **Pengaturan panjang array di Vue 2:** `arr.length = 0` tidak memicu reactivity
-   **Mutasi props secara langsung:** Pola yang harus dihindari dalam pengembangan
-   **Masalah timing template ref:** Mengakses refs sebelum komponen di-mount

### Data Flow Between Components

Vue.js menerapkan pola **aliran data searah (unidirectional data flow)**:

**Parent ke Child (Props):**

-   Menggunakan props untuk mengirim data dari parent ke child component
-   Props bersifat read-only di child component
-   Binding menggunakan v-bind atau shorthand `:`

**Child ke Parent (Events):**

-   Menggunakan `$emit` untuk mengirim event dari child ke parent
-   Parent mendengarkan event menggunakan v-on atau shorthand `@`
-   Bisa mengirim data sebagai payload event

**Binding dua arah (v-model):**

-   Kombinasi props dan events untuk two-way data binding
-   Menggunakan `value` prop dan `update:value` event
-   Shorthand untuk pattern yang umum digunakan

**Slot Props untuk komunikasi yang fleksibel:**

-   Memungkinkan parent mengakses data dari child melalui slot
-   Child menyediakan data melalui slot props
-   Parent bisa menggunakan data tersebut dalam template

**Komunikasi antar sibling:**

-   Memanfaatkan parent bersama sebagai perantara
-   Event bus (Vue 2) atau mekanisme provide/inject
-   Implementasi state management (Vuex/Pinia)

**Komunikasi komponen tingkat dalam:**

-   **Provide/Inject:** Untuk dependency injection di seluruh pohon komponen
-   **Event bus:** Komunikasi global (batasi penggunaan)
-   **State management:** Store terpusat untuk aplikasi dengan kompleksitas tinggi

### Common Memory Leak Causes and Solutions

**Penyebab utama yang paling sering:**

1.  **Event listeners yang tidak dibersihkan:**
    
    -   Event listeners yang ditambahkan ke window, document, atau DOM elements
    -   Solusi: Hapus event listeners di lifecycle hook `beforeUnmount`
2.  **Timer yang tidak dihentikan:**
    
    -   `setInterval`, `setTimeout` yang tidak dibersihkan
    -   Solusi: Simpan reference dan gunakan `clearInterval`/`clearTimeout`
3.  **Referensi DOM yang masih tersimpan:**
    
    -   Menyimpan referensi DOM elements di data atau variables
    -   Solusi: Set ke null saat cleanup atau gunakan template refs
4.  **Watcher reaktif yang tidak dihentikan:**
    
    -   Watchers yang dibuat secara manual tidak otomatis dihapus
    -   Solusi: Simpan unwatch function dan panggil saat cleanup
5.  **Instance library pihak ketiga tidak dimusnahkan:**
    
    -   Chart libraries, map instances, atau plugins lain
    -   Solusi: Call destroy/remove methods di beforeUnmount
6.  **Komponen rekursif tanpa kondisi penghentian:**
    
    -   Komponen yang merender dirinya sendiri tanpa batas
    -   Solusi: Tambahkan kondisi untuk menghentikan rekursi
7.  **Global mixins yang tidak dibersihkan:**
    
    -   State global yang tidak dibersihkan saat komponen dihancurkan
    -   Solusi: Cleanup global state di lifecycle hooks
8.  **Akumulasi komponen keep-alive:**
    
    -   Komponen yang di-cache oleh keep-alive menumpuk
    -   Solusi: Gunakan include/exclude untuk membatasi caching
9.  **Referensi sirkular dalam data:**
    
    -   Objek yang saling mereferensi satu sama lain
    -   Solusi: Hindari atau putuskan referensi saat cleanup

### State Management Solutions Used

Untuk aplikasi Vue.js, solusi manajemen state yang biasa diimplementasikan:

**Pinia (Rekomendasi untuk Vue 3):**

-   Arsitektur modern dengan dukungan TypeScript yang baik
-   Integrasi DevTools yang superior
-   API yang lebih simpel dibanding Vuex
-   Code splitting yang otomatis
-   Dukungan hot module replacement

**Vuex (Legacy untuk Vue 2/3):**

-   Ekosistem yang sudah matang
-   Fitur time-travel debugging
-   Aliran data searah yang ketat
-   Ekosistem plugin yang luas

**Alternatif sederhana:**

-   **Provide/Inject** untuk aplikasi berukuran kecil
-   **Composables dengan reactive()** untuk kompleksitas menengah
-   **Event bus** untuk komunikasi sederhana antar komponen
-   **Browser storage** (localStorage, sessionStorage) untuk persistensi
-   **URL/Router state** untuk state yang perlu dapat di-bookmark

### Pre-rendering vs Server-Side Rendering

**Pre-rendering (Generasi Situs Statis):**

-   HTML dibuat pada **waktu build**
-   HTML identik disajikan untuk semua pengguna
-   Pemuatan awal yang cepat, ideal untuk konten statis
-   Kemampuan dinamis yang terbatas
-   Cocok untuk: Blog, dokumentasi, halaman landing

**Server-Side Rendering (SSR):**

-   HTML dibuat pada **waktu request** di server
-   Konten dinamis untuk setiap permintaan
-   Performa SEO dan pemuatan awal yang superior
-   Konfigurasi dan kebutuhan hosting yang lebih rumit
-   Aplikasi lengkap berjalan di sisi server
-   Cocok untuk: Aplikasi dinamis, e-commerce, dashboard

**Client-Side Rendering (CSR):**

-   HTML dibuat di browser
-   JavaScript dimuat terlebih dahulu, kemudian render konten
-   Interaktivitas penuh setelah loading selesai
-   Optimasi SEO yang kurang optimal
-   Cocok untuk: SPA, aplikasi internal

**Hybrid Rendering:**

-   Kombinasi SSR dengan CSR
-   Loading awal dari server, navigasi selanjutnya client-side
-   Mengambil keunggulan dari kedua pendekatan

**Perbedaan utama:**

-   **Waktu eksekusi:** Build time vs Request time vs Client time
-   **Jenis konten:** Statis vs Dinamis vs Interaktif
-   **Tingkat kompleksitas:** Deploy sederhana vs Infrastruktur server vs Optimasi client
-   **Performa:** Serving cepat vs Fleksibilitas vs Interaktivitas
-   **SEO:** Excellent vs Good vs Poor (tanpa optimasi)
-   **Hydration:** Tidak ada vs Ada vs Tidak diperlukan

**Proses Hydration:**

-   Server mengirimkan HTML statis
-   Client "mengaktifkan" HTML dengan JavaScript
-   Event listeners dan reaktivitas ditambahkan
-   Komponen menjadi interaktif

**Pertimbangan Client-side Routing:**

-   SSR: Penanganan route di server dan client
-   Pre-rendering: Client-side routing setelah initial load
-   CSR: Full client-side routing

## Website Security Best Practises

### HTTPS (SSL/TLS Encryption)

HTTPS adalah fondasi keamanan web modern yang mengenkripsi semua komunikasi antara browser dan server.

**Mengapa Penting:**

-   **Proteksi data in-transit** - Mencegah man-in-the-middle attacks
-   **Data integrity** - Memastikan data tidak dimodifikasi selama transmisi
-   **Authentication** - Memverifikasi identitas server
-   **SEO boost** - Google memberikan ranking preference untuk HTTPS sites
-   **User trust** - Browser menampilkan warning untuk HTTP sites

**Implementasi Best Practices:**

-   Gunakan certificate dari Certificate Authority (CA) terpercaya
-   Enable Perfect Forward Secrecy (PFS)
-   Implement HSTS dengan preload list
-   Regular certificate renewal dan monitoring
-   Use strong cipher suites dan disable weak protocols
-   Redirect semua HTTP traffic ke HTTPS
-   Configure secure headers untuk enhanced security

### CSRF (Cross-Site Request Forgery) Protection

CSRF attacks memaksa user yang sudah terautentikasi untuk melakukan aksi yang tidak diinginkan pada aplikasi web.

**Mengapa Berbahaya:**

-   **Unauthorized actions** - Attacker bisa melakukan aksi atas nama user
-   **Silent attacks** - User tidak menyadari aksi sedang dilakukan
-   **Privilege escalation** - Bisa menggunakan privileges user yang sedang login
-   **Data manipulation** - Transfer uang, ubah password, delete data

**Cara Kerja CSRF Attack:**

1.  User login ke aplikasi legitimate (bank.com)
2.  User mengunjungi malicious site (evil.com)
3.  Malicious site mengirim request ke bank.com menggunakan session user
4.  Bank.com memproses request karena user terautentikasi

**Protection Methods:**

-   **CSRF Tokens** - Unique tokens untuk setiap form/request
-   **SameSite Cookies** - Restrict cookie sending in cross-site requests
-   **Double Submit Cookies** - Token di cookie dan header/form
-   **Referrer Header Validation** - Verify request origin
-   **Custom Headers** - Require specific headers untuk AJAX requests

### Security Headers

Security headers memberikan instruksi kepada browser tentang cara menangani konten dengan aman.

**Content Security Policy (CSP):**

-   Mengontrol resource mana yang boleh dimuat oleh browser
-   Mencegah XSS attacks dengan membatasi script sources
-   Directive untuk mengontrol berbagai jenis resources
-   Report violations untuk monitoring

**X-Frame-Options:**

-   Mencegah clickjacking attacks
-   Mengontrol apakah halaman bisa di-embed dalam frame
-   Options: DENY, SAMEORIGIN, ALLOW-FROM

**X-Content-Type-Options:**

-   Mencegah MIME type sniffing attacks
-   Memaksa browser menggunakan declared content type
-   Value: nosniff

**Referrer-Policy:**

-   Mengontrol informasi referrer yang dikirim
-   Options: no-referrer, same-origin, strict-origin-when-cross-origin

**Permissions-Policy:**

-   Mengontrol browser features yang bisa diakses
-   Disable features seperti camera, microphone, geolocation

### API Security

API security melindungi endpoints dari unauthorized access dan abuse.

**Authentication Methods:**

-   **JWT (JSON Web Tokens)** - Stateless authentication dengan signed tokens
-   **OAuth 2.0** - Industry standard untuk authorization
-   **API Keys** - Simple authentication untuk service-to-service
-   **Basic Auth** - Username/password untuk simple scenarios

**Security Measures:**

-   **Rate Limiting** - Prevent abuse dan DoS attacks
-   **Input Validation** - Validate semua parameters dan payloads
-   **HTTPS Only** - Encrypt all API communications
-   **CORS Configuration** - Restrict cross-origin requests
-   **API Versioning** - Maintain backward compatibility
-   **Request/Response Logging** - Audit trail untuk security events
-   **Error Handling** - Don't expose sensitive information

### Input Validation & Sanitization

Input validation adalah proses memverifikasi dan membersihkan semua data yang diterima dari user sebelum diproses oleh aplikasi.

**Mengapa Penting:**

-   **SQL Injection Prevention** - Mencegah malicious SQL queries
-   **XSS Protection** - Mencegah script injection ke dalam pages
-   **Data Integrity** - Memastikan data sesuai format yang diharapkan
-   **Buffer Overflow Prevention** - Mencegah memory corruption attacks
-   **Business Logic Protection** - Memastikan data masuk akal secara bisnis

**Types of Validation:**

-   **Server-Side Validation** - Primary validation yang tidak bisa di-bypass
-   **Client-Side Validation** - UX improvement tapi tidak reliable untuk security
-   **Data Type Validation** - Ensure correct data types
-   **Length Validation** - Prevent buffer overflows
-   **Format Validation** - Email, phone, URL patterns
-   **Range Validation** - Numeric ranges, date ranges
-   **Whitelist Validation** - Only allow expected values

**SQL Injection Prevention:**

-   Use parameterized queries/prepared statements
-   Never concatenate user input directly into SQL
-   Use ORM dengan built-in protection
-   Validate dan escape input
-   Use least privilege database accounts

**XSS Prevention:**

-   HTML entity encoding untuk output
-   Input sanitization untuk remove/escape HTML
-   Content Security Policy implementation
-   Use template engines dengan auto-escaping
-   Validate dan sanitize rich text input

**File Upload Security:**

-   Validate file types dan extensions
-   Check file content headers
-   Limit file sizes
-   Store uploads outside web root
-   Scan untuk malware
-   Generate safe filenames

**Rate Limiting:**

-   Prevent brute force attacks
-   Limit requests per IP/user
-   Different limits untuk different endpoints
-   Progressive delays untuk repeated violations

### Authentication & Session Management

Authentication memverifikasi identitas user, sedangkan session management mengelola state user yang sudah terautentikasi.

**Strong Password Policies:**

-   Minimum length requirements (8+ characters)
-   Complexity requirements (uppercase, lowercase, numbers, symbols)
-   Password strength meters untuk user guidance
-   Prevent common passwords
-   Password history untuk prevent reuse
-   Regular password expiration policies

**Password Security:**

-   Strong hashing algorithms (bcrypt, Argon2, PBKDF2)
-   Salt untuk prevent rainbow table attacks
-   Proper salt generation dan storage
-   Secure password reset mechanisms
-   Account lockout setelah failed attempts

**Session Management:**

-   Secure session storage (server-side)
-   Session timeout untuk inactive users
-   Session regeneration setelah privilege changes
-   Secure cookie configuration (HttpOnly, Secure, SameSite)
-   Proper session invalidation pada logout
-   Session monitoring untuk concurrent sessions

**Multi-Factor Authentication (MFA):**

-   TOTP (Time-based One-Time Password)
-   SMS verification (less secure)
-   Hardware tokens
-   Biometric authentication
-   Backup codes untuk recovery
-   Risk-based authentication

**Account Security Features:**

-   Account lockout policies
-   Login attempt monitoring
-   Suspicious activity detection
-   Email notifications untuk security events
-   Password change notifications
-   Device/location tracking
-   Secure account recovery processes

## Website Performance Best Practises

### Image Optimization

Images biasanya merupakan resource terbesar di web pages, optimasi yang proper bisa dramatically improve performance.

**Mengapa Penting:**
- **Largest Impact on Page Size** - Images often account for 60%+ of page weight
- **Loading Performance** - Slow images delay LCP dan visual completeness
- **Bandwidth Usage** - Especially important untuk mobile users
- **User Experience** - Fast loading images improve perceived performance

**Optimization Techniques:**
- **Modern Image Formats** - WebP, AVIF untuk better compression
- **Responsive Images** - Serve appropriate sizes dengan srcset dan sizes
- **Lazy Loading** - Load images only when needed (below-the-fold)
- **Image Compression** - Balance quality vs file size
- **Proper Dimensions** - Avoid browser resizing dengan correct width/height
- **Image CDN** - Global distribution untuk faster delivery
- **Art Direction** - Different images untuk different screen sizes

### Minimize HTTP Requests

Setiap HTTP request menambah latency, mengurangi jumlah requests adalah salah satu optimasi paling efektif.

**Mengapa Penting:**
- **Network Latency** - Each request adds round-trip time
- **Connection Limits** - Browsers limit concurrent connections
- **Mobile Performance** - High-latency networks amplify the problem
- **Resource Overhead** - Each request has protocol overhead

**Reduction Strategies:**
- **Bundle CSS/JavaScript** - Combine multiple files into fewer bundles
- **CSS Sprites** - Combine small images into single sprite sheets
- **Inline Small Resources** - Embed small CSS/JS directly in HTML
- **HTTP/2 Server Push** - Proactively send critical resources
- **Reduce Third-Party Requests** - Audit dan minimize external dependencies
- **Resource Consolidation** - Combine similar functionality into single files

### Leverage Browser Caching

Proper caching strategy mengurangi repeat downloads dan improves return visit performance.

**Mengapa Penting:**
- **Repeat Visit Performance** - Cached resources load instantly
- **Bandwidth Savings** - Reduce server load dan user data usage
- **Offline Capability** - Enable offline functionality dengan service workers
- **User Experience** - Faster navigation dan interaction

**Caching Strategies:**
- **Cache Headers** - Set appropriate max-age untuk different resource types
- **Versioning/Fingerprinting** - Use content hashes untuk cache busting
- **CDN Caching** - Global edge caching untuk static resources
- **Service Workers** - Programmatic caching control
- **Browser Cache API** - Fine-grained caching control
- **Cache Invalidation** - Proper strategies untuk updating cached content

### Code Splitting & Lazy Loading

Loading only what's needed when it's needed improves initial page load performance.

**Mengapa Penting:**
- **Reduced Initial Bundle Size** - Faster time to interactive
- **Better Resource Utilization** - Load resources on-demand
- **Memory Efficiency** - Don't load unused code
- **Progressive Loading** - Improve perceived performance

**Implementation Strategies:**
- **Route-Based Splitting** - Split code by application routes
- **Component-Based Splitting** - Lazy load heavy components
- **Dynamic Imports** - Load modules when needed
- **Intersection Observer** - Detect when content enters viewport
- **Progressive Enhancement** - Core functionality first, enhancements later
- **Bundle Analysis** - Identify dan eliminate unused code

### Database Optimization (Indexing)

Database performance directly impacts application response times dan overall user experience.

**Mengapa Penting:**
- **Query Performance** - Proper indexes dramatically reduce query execution time
- **Scalability** - Efficient database operations handle more concurrent users
- **Resource Usage** - Optimized queries use less CPU dan memory
- **User Experience** - Fast database responses improve perceived performance

**Indexing Strategies:**
- **Primary Indexes** - Automatically created untuk primary keys
- **Composite Indexes** - Multiple columns untuk complex queries
- **Partial Indexes** - Index subset of rows based on conditions
- **Unique Indexes** - Enforce uniqueness dan improve lookup performance
- **Full-Text Indexes** - Optimize text search queries
- **Index Maintenance** - Regular analysis dan rebuilding untuk optimal performance

**Query Optimization:**
- **Avoid N+1 Queries** - Use joins atau batch queries instead of multiple round trips
- **Use LIMIT** - Don't fetch more data than needed
- **Query Profiling** - Analyze execution plans untuk identify bottlenecks
- **Connection Pooling** - Reuse database connections efficiently
- **Caching Layer** - Redis atau Memcached untuk frequently accessed data
- **Database Normalization** - Proper table structure untuk efficient storage dan retrieval

##  .NET 
```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text.RegularExpressions;

public class Program
{
    public static Dictionary<string, int> CountWordFrequency(string input)
    {
        if (string.IsNullOrEmpty(input))
            return new Dictionary<string, int>();
        
        string cleanedInput = Regex.Replace(input.ToLower(), @"[^\w\s]", "");
        
        string[] words = cleanedInput.Split(new char[] { ' ', '\t', '\n', '\r' }, 
                                          StringSplitOptions.RemoveEmptyEntries);
        
        var wordFrequency = words
            .GroupBy(word => word)
            .ToDictionary(group => group.Key, group => group.Count());
        
        return wordFrequency;
    }
    
    public static void Main()
    {
        string testString = "Four, One two two three Three three four  four   four";
        
        var result = CountWordFrequency(testString);
        
        Console.WriteLine("Word Frequency Count:");
        foreach (var kvp in result.OrderBy(x => x.Key))
        {
            Console.WriteLine($"{kvp.Key} => {kvp.Value}");
        }
    }
}
```

## Tools (Rate yourself 1 to 5)

| Tool                 | Rating |
|----------------------|--------|
| Git                  | 4      |
| Redis                | 3      |
| VSCode / JetBrains   | 4      |
| Linux                | 3      |
| AWS EC2              | 3      |
| AWS Lambda           | 3      |
| AWS RDS              | 4      |
| AWS CloudWatch       | 4      |
| AWS S3               | 4      |
| Unit testing         | 3      |
| Kanban boards        | 4      |
