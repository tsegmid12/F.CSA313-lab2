# Лаборатори №2: k6 ашиглан гүйцэтгэл хэмжих

**Хичээл:** F.CSA313 — Программ хангамжийн чанарын баталгаа ба туршилт  
**Оюутан:** Э.Цэгмид  
**Оюутны код:** B232270107  
**Туршилтын орчин:** Windows 11, PowerShell 7, Visual Studio Code

**k6 version:** v2.2.0
commit: 00a9a1b7f5
Go: go1.26.5
OS/Architecture: windows/amd64

## 1. Лабораторийн зорилго

Grafana k6 ашиглан `https://test.k6.io` веб серверийн гүйцэтгэлийг хэмжиж, ачаалал нэмэгдэхэд дараах үзүүлэлтүүд хэрхэн өөрчлөгдөхийг шалгав:

- latency буюу хариу өгөх хугацаа;
- throughput буюу секундэд боловсруулах хүсэлтийн тоо;
- HTTP error rate;
- k6 check болон threshold-ийн үр дүн.

Туршилтад 5, 30, 100 Virtual Users (VU), мөн шаталсан ачааллын scenario ашигласан.

## 2. Ашигласан технологи

| Технологи | Зориулалт |
|---|---|
| Grafana k6 | Load/performance testing |
| JavaScript | k6 тестийн скрипт |
| Windows 11 + PowerShell 7 | Ажиллуулах орчин |
| Git/GitHub | Хувилбарын удирдлага |
| Visual Studio Code | Код засварлагч |

Тестийн target URL: `https://test.k6.io`  
k6 хувилбарын гаралт үр дүнгийн файлд хадгалагдаагүй.

## 3. Repository бүтэц

```text
F.CSA313-lab2/
├── scripts/
│   ├── script.js           # 5 VU, 30 секундийн baseline
│   ├── load-test.js        # 5 VU, 1 минутын үндсэн тест
│   ├── stages.js           # 5 → 30 → 100 → 0 VU шатлал
│   ├── threshold-pass.js   # p(95) < 368 ms, error rate < 1%
│   └── threshold-fail.js   # p(95) < 50 ms, зориудаар FAIL болгох тест
├── results/
│   ├── script.txt
│   ├── run-05vu.txt
│   ├── run-30vu.txt
│   ├── run-100vu.txt
│   ├── stages.txt
│   ├── pass.txt
│   └── fail.txt
├── screenshots/
│   └── stages-summery.png
└── README.md
```

Бүх скрипт `https://test.k6.io` руу GET хүсэлт илгээж, HTTP status `200` болсон эсэхийг check-ээр шалгана. Нэг iteration бүрийн дараа `1s` sleep хийнэ.

## 4. Тест ажиллуулах

PowerShell дээр repository-ийн үндсэн хавтсаас:

```powershell
k6 run scripts/script.js
k6 run --vus 5 scripts/load-test.js
k6 run --vus 30 scripts/load-test.js
k6 run --vus 100 scripts/load-test.js
k6 run scripts/stages.js
k6 run scripts/threshold-pass.js
k6 run scripts/threshold-fail.js
```

Үр дүнг файлд хадгалах жишээ:

```powershell
k6 run scripts/load-test.js | Tee-Object results/run-05vu.txt
```

## 5. Baseline үр дүн

Baseline нь `scripts/script.js`-ийг 5 VU, 30 секунд ажиллуулсан үр дүн бөгөөд бүрэн гаралт нь [`results/script.txt`](results/script.txt)-д байна.

| Үзүүлэлт | Үр дүн |
|---|---:|
| Virtual Users | 5 |
| Duration | 30 секунд |
| Average latency | 177.45 ms |
| p90 latency | 233.38 ms |
| p95 latency | 245.30 ms |
| Throughput | 7.009757 req/s |
| HTTP requests | 224 |
| HTTP error rate | 0% (0/224) |
| Checks | 100% (112/112) |

## 6. VU түвшний харьцуулалт

`results/run-05vu.txt`, `results/run-30vu.txt`, `results/run-100vu.txt` файлуудын summary-ээс:

| Үзүүлэлт | 5 VU | 30 VU | 100 VU |
|---|---:|---:|---:|
| Duration | 1 мин | 1 мин | 1 мин |
| Average latency | 142.66 ms | 152.06 ms | 148.50 ms |
| p90 latency | 225.59 ms | 226.90 ms | 225.30 ms |
| p95 latency | 228.07 ms | 231.26 ms | 228.30 ms |
| Throughput | 7.614472 req/s | 44.951237 req/s | 150.7634 req/s |
| HTTP requests | 466 | 2,754 | 9,236 |
| HTTP error rate | 0% | 0% | 0% |
| Checks | 100% (233/233) | 100% (1,377/1,377) | 100% (4,618/4,618) |

### Тайлбар

5 VU-ээс 100 VU хүртэл өсгөхөд throughput 7.61-ээс 150.76 req/s болж мэдэгдэхүйц нэмэгдсэн. Харин p95 latency 228.07–231.26 ms-ийн хооронд байсан бөгөөд гурван туршилтад HTTP error rate 0% гарсан. Энэ нь тухайн туршилтын богино хугацаа, target орчин болон сүлжээний нөхцөлд 100 VU хүртэл тогтвортой үр дүн үзүүлснийг харуулна. Энэ дүгнэлтийг production capacity-ийн баталгаа гэж үзэхгүй.

## 7. Stages туршилт

`scripts/stages.js` дараах шатлалаар ажилласан:

| Үе шат | Хугацаа | Target VU |
|---|---:|---:|
| Warm-up | 30 секунд | 5 |
| Ramp-up | 1 минут | 30 |
| Peak | 30 секунд | 100 |
| Ramp-down | 30 секунд | 0 |

Нийт stage хугацаа 2 минут 30 секунд. Бодит summary (`results/stages.txt`):

| Үзүүлэлт | Үр дүн |
|---|---:|
| HTTP requests | 7,084 |
| Throughput | 46.951921 req/s |
| Average latency | 142.35 ms |
| p90 latency | 223.39 ms |
| p95 latency | 226.21 ms |
| HTTP error rate | 0% |
| Checks | 100% (3,542/3,542) |

Дэлгэцийн summary-г [`screenshots/stages-summery.png`](screenshots/stages-summery.png)-оос харж болно.

## 8. Threshold ба SLO

Бүх threshold тестэд error rate-ийн босго `http_req_failed: rate<0.01`, өөрөөр хэлбэл **1%-иас бага** байхаар тохируулсан.

### PASS тест

[`scripts/threshold-pass.js`](scripts/threshold-pass.js) файлд бодитоор:

```text
http_req_duration: p(95)<368
http_req_failed:   rate<0.01
```

Үр дүн (`results/pass.txt`):

- p95 = 229.74 ms;
- error rate = 0%;
- хоёр threshold хангагдсан тул тест PASS болсон.

### FAIL тест

[`scripts/threshold-fail.js`](scripts/threshold-fail.js) файлд p95 босгыг зориудаар `50 ms` болгож бууруулсан.

Үр дүн (`results/fail.txt`):

- p95 = 228.05 ms;
- error rate = 0%;
- `228.05 > 50` тул `http_req_duration` threshold зөрчигдөж, k6 FAIL болсон.

### Baseline-д суурилсан SLO тооцоо

Baseline тестийн p95 response time нь **245.30 ms** байв. Үүнийг 1.5 дахин өсгөж SLO босгыг тооцооллоо.

```text
245.30 × 1.5 = 367.95 ms ≈ 368 ms
```

Иймээс baseline-д суурилсан SLO шаардлага:

- Response time: `p(95) < 368 ms`
- Error rate: `rate < 1%`

`threshold-pass.js` файлд дээрх SLO босгыг ашигласан. Харин `threshold-fail.js` файлд `p(95) < 50 ms` гэсэн зориудаар биелэх боломжгүй хатуу босго тавьж, threshold зөрчигдөх үеийн үр дүнг шалгасан.

## 9. Дүгнэлт

1. k6 ашиглан baseline, тогтмол VU, stages болон threshold тестүүдийг амжилттай ажиллуулсан.
2. 100 VU үед хамгийн өндөр throughput болох 150.7634 req/s хэмжигдсэн.
3. 5, 30, 100 VU түвшинд p95 latency ойролцоогоор 228–231 ms, HTTP error rate 0% байв.
4. Stages тестийн p95 latency 226.21 ms, нийт хүсэлт 7,084 байв.
5. Threshold PASS/FAIL тестүүд нь performance босгыг k6 автоматаар шалгаж чаддагийг харуулсан.
6. Эдгээр үр дүн нь зөвхөн тухайн `test.k6.io`, хугацаа, сүлжээ болон ажиллуулсан орчны нөхцөлд хамаарна.
7. Baseline p95 latency дээр үндэслэн SLO-г 368 ms, error rate-ийг 1%-иас бага байхаар тогтоосон.
8. PASS болон FAIL threshold-ийн үр дүн нь гүйцэтгэлийн босгыг CI/CD quality gate хэлбэрээр ашиглаж болохыг баталсан.

## 10. Ёс зүй

Бүх тестийг k6-ийн зөвшөөрөгдсөн дадлагын сайт болох `https://test.k6.io` руу чиглүүлсэн. Зөвшөөрөлгүй гуравдагч этгээдийн системд load test явуулаагүй.

## 11. Үр дүнгийн файлууд

- [`results/script.txt`](results/script.txt) — 5 VU, 30 секундийн baseline
- [`results/run-05vu.txt`](results/run-05vu.txt) — 5 VU, 1 минут
- [`results/run-30vu.txt`](results/run-30vu.txt) — 30 VU, 1 минут
- [`results/run-100vu.txt`](results/run-100vu.txt) — 100 VU, 1 минут
- [`results/stages.txt`](results/stages.txt) — шаталсан ачаалал
- [`results/pass.txt`](results/pass.txt) — threshold PASS
- [`results/fail.txt`](results/fail.txt) — threshold FAIL
