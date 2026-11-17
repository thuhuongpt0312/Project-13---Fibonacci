# Project 13: Dãy Fibonacci và Tỉ lệ vàng (Kotlin Compose)

Kotlin from Scratch – Part II: Chapter 4 – Project 13: Code the Fibonacci Sequence (Golden Ratio & Fibonacci Spiral). Phiên bản Android dùng Kotlin và Jetpack Compose.

## Thông tin sinh viên

| Thông tin | Nội dung |
|---|---|
| Họ và tên | Phan Thị Thu Hương |
| MSSV | 067306005888 |
| Lớp | 012012103402 |

## Cấu trúc repository

```
Project-13---Fibonacci/
├── SourceCode/
│   └── MainActivity.kt   # Toàn bộ mã nguồn ứng dụng
├── Slide/                # Slide thuyết trình
└── README.md
```

## Chức năng

- **Màn hình danh sách (`LazyColumn`):** hiển thị F(n) và tỉ số F(n) / F(n−1); chọn N từ 3 đến 20 bằng thanh trượt.
- **Màn hình xoắn ốc (`Canvas`):** vẽ các ô vuông có cạnh Fibonacci, ghi số trong ô và vẽ đường xoắn ốc bằng các cung 1/4 đường tròn.
- **Điều hướng:** chuyển giữa hai màn hình bằng Navigation Compose, truyền N qua route `spiral/{n}`.

## Cách chạy

1. Tạo project **Empty Activity (Compose)** trong Android Studio, package `com.example.fibonacci`.
2. Chép `SourceCode/MainActivity.kt` thay cho file `MainActivity.kt` của project.
3. Thêm vào `build.gradle.kts` (module app) rồi Sync:
   ```kotlin
   implementation("androidx.navigation:navigation-compose:2.8.5")
   implementation("androidx.compose.material:material-icons-core:1.7.5")
   ```
4. Chạy ứng dụng trên máy ảo hoặc điện thoại thật.

---

## 1. Cơ sở toán học

### 1.1. Dãy Fibonacci

Dãy được xác định bằng một quy tắc duy nhất (bản của ứng dụng bắt đầu từ 1, 1):

```
F(1) = F(2) = 1
F(n) = F(n−1) + F(n−2)    với n ≥ 3
```

Các số hạng đầu: 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, ...

Số trong ngoặc là **vị trí**, còn kết quả là **giá trị**. Ví dụ F(10) = 55 nghĩa là số hạng thứ 10 của dãy có giá trị 55 (bằng 34 + 21).

### 1.2. Tỉ lệ vàng φ

```
φ = (1 + √5) / 2 ≈ 1.618034
```

Gọi **r(n) = F(n) / F(n−1)** là tỉ số hai số liền nhau. Khi n tăng, r(n) tiến về φ.

**Vì sao r(n) tiến về φ?** Vì F(n) = F(n−1) + F(n−2) nên

```
r(n) = F(n)/F(n−1) = 1 + F(n−2)/F(n−1) = 1 + 1 / r(n−1)
```

Nếu r(n) tiến về một giới hạn r thì r = 1 + 1/r, suy ra r² − r − 1 = 0. Nghiệm dương của phương trình này là (1 + √5) / 2, tức φ.

### 1.3. Số liệu tỉ số lấy từ ứng dụng (N = 10)

| Dòng | Giá trị F(n) | Phép chia | Tỉ số r(n) | Lệch so với φ |
|---|---|---|---|---|
| F(1) | 1 | không có số trước | — | — |
| F(2) | 1 | 1 / 1 | 1.000000 | −0.618 |
| F(3) | 2 | 2 / 1 | 2.000000 | +0.382 |
| F(4) | 3 | 3 / 2 | 1.500000 | −0.118 |
| F(5) | 5 | 5 / 3 | 1.666667 | +0.049 |
| F(6) | 8 | 8 / 5 | 1.600000 | −0.018 |
| F(7) | 13 | 13 / 8 | 1.625000 | +0.007 |
| F(8) | 21 | 21 / 13 | 1.615385 | −0.003 |
| F(9) | 34 | 34 / 21 | 1.619048 | +0.001 |
| F(10) | 55 | 55 / 34 | 1.617647 | −0.0004 |

**Nhận xét về tỉ số:**

- **F(3) có tỉ số 2.000000** vì 2 / 1 = 2. Ở đầu dãy các số còn rất nhỏ nên phép chia dao động mạnh; đây là hiện tượng bình thường, không phải lỗi.
- Tỉ số **dao động quanh φ**: lúc cao hơn (2.0, 1.667, 1.625, 1.619), lúc thấp hơn (1.0, 1.5, 1.6, 1.615, 1.6176).
- Biên độ dao động **nhỏ dần**, mỗi bước sai lệch giảm còn khoảng 0.382 lần (bằng 1/φ²) bước trước.
- Từ khoảng F(6) sai lệch dưới 0.02; từ F(8) tỉ số đã khớp φ ở hai chữ số đầu (1.61...); đến F(10) sai lệch dưới 0.001.

---

## 2. Giải thích các hàm và độ phức tạp

Ký hiệu: **n** là số phần tử N được chọn (3 ≤ n ≤ 20).

### 2.1. `fibonacci(count)`: sinh dãy

```kotlin
fun fibonacci(count: Int): List<Long> {
    val list = mutableListOf(1L, 1L)
    while (list.size < count)
        list.add(list[list.size - 1] + list[list.size - 2])
    return list.take(count)
}
```

- **Chức năng:** trả về `count` số Fibonacci đầu tiên, bắt đầu từ 1, 1.
- **Cách chạy:** khởi tạo danh sách hai phần tử (1, 1), sau đó lặp: thêm tổng hai phần tử cuối cho đến khi đủ số lượng. `take(count)` cắt đúng `count` phần tử, phòng khi `count` nhỏ hơn 2.
- **Kiểu `Long`:** chính xác đến F(92); từ F(93) bị tràn số. Ứng dụng chỉ cho N ≤ 20 nên an toàn.
- **Độ phức tạp thời gian: O(n).** Vòng `while` chạy khoảng n − 2 lần, mỗi lần truy cập danh sách và thêm phần tử đều là O(1) (thêm vào cuối có chi phí trung bình O(1)). `take` sao chép tối đa n phần tử, cũng là O(n).
- **Độ phức tạp bộ nhớ: O(n)** để lưu danh sách kết quả.

**So sánh các cách tính:**

| Cách tính | Thời gian | Nhận xét |
|---|---|---|
| Vòng lặp (dùng trong ứng dụng) | O(n) | Đơn giản, nhanh, dễ kiểm chứng |
| Đệ quy trực tiếp F(n) = F(n−1) + F(n−2) | xấp xỉ O(φⁿ), thường ghi cận trên O(2ⁿ) | Tính lặp lại các số con nên rất chậm khi n lớn |
| Đệ quy có nhớ (memoization) | O(n) | Khắc phục việc tính lặp, tốn thêm bộ nhớ |

### 2.2. `Piece`: dữ liệu một ô vuông

```kotlin
data class Piece(
    val l: Float, val t: Float, val r: Float, val b: Float,
    val cx: Float, val cy: Float
) {
    val side get() = r - l
}
```

- Lưu một ô vuông gồm cạnh trái `l`, trên `t`, phải `r`, dưới `b` và tâm cung tròn `(cx, cy)` nằm trong ô đó.
- Thuộc tính `side = r − l` là độ dài cạnh ô; cũng là bán kính của cung tròn.
- Mọi thao tác đều O(1).

### 2.3. `buildPieces(fibs)`: xếp ô vuông và tính tâm cung

```kotlin
fun buildPieces(fibs: List<Long>): List<Piece> {
    data class Box(val l: Float, val t: Float, val r: Float, val b: Float)

    val boxes = mutableListOf(Box(0f, 0f, 1f, 1f))
    var bb = boxes[0] // khung bao của tất cả ô đã đặt
    for (i in 1 until fibs.size) {
        val s = fibs[i].toFloat()
        val box = when ((i - 1) % 4) {
            0 -> Box(bb.r, bb.t, bb.r + s, bb.t + s)      // sang phải
            1 -> Box(bb.l, bb.t - s, bb.l + s, bb.t)      // lên trên
            2 -> Box(bb.l - s, bb.b - s, bb.l, bb.b)      // sang trái
            else -> Box(bb.r - s, bb.b, bb.r, bb.b + s)   // xuống dưới
        }
        boxes.add(box)
        bb = Box(min(bb.l, box.l), min(bb.t, box.t), maxOf(bb.r, box.r), maxOf(bb.b, box.b))
    }

    return boxes.mapIndexed { i, bx ->
        if (i == 0) {
            Piece(bx.l, bx.t, bx.r, bx.b, cx = bx.r, cy = bx.t)
        } else {
            val d = (i - 1) % 4   // hướng đặt ô hiện tại
            val dn = i % 4        // hướng đặt ô kế tiếp
            val prevVal = when (d) { 0 -> bx.l; 1 -> bx.b; 2 -> bx.r; else -> bx.t }
            val nextVal = when (dn) { 0 -> bx.r; 1 -> bx.t; 2 -> bx.l; else -> bx.b }
            val cx = if (d % 2 == 0) prevVal else nextVal
            val cy = if (d % 2 == 0) nextVal else prevVal
            Piece(bx.l, bx.t, bx.r, bx.b, cx, cy)
        }
    }
}
```

- **Chức năng:** từ dãy Fibonacci, tính vị trí từng ô vuông và tâm cung tròn tương ứng.
- **Bước 1, xếp ô:** ô đầu có cạnh 1 tại (0, 0)–(1, 1). Biến `bb` là khung bao của mọi ô đã đặt. Ô thứ i có cạnh `fibs[i]`, hướng đặt là `(i − 1) % 4`: 0 sang phải, 1 lên trên, 2 sang trái, 3 xuống dưới. Ô mới dính sát khung bao, sau đó khung bao được mở rộng bằng `min` và `max`. Lưu ý trục y của màn hình hướng xuống nên "lên trên" nghĩa là y giảm.
- **Bước 2, tìm tâm cung:** mỗi ô dính ô trước ở một cạnh và dính ô sau ở một cạnh khác. Tâm cung là giao của hai đường chứa hai cạnh đó. `prevVal` là toạ độ cạnh chung với ô trước, `nextVal` là toạ độ cạnh chung với ô sau. Hướng đặt chẵn hay lẻ quyết định giá trị nào là x, giá trị nào là y. Nhờ vậy các cung ở hai ô liền nhau có chung điểm nối và cùng tiếp tuyến, tạo thành đường xoắn ốc liền mạch.
- **Độ phức tạp thời gian: O(n).** Vòng `for` chạy n − 1 lần, mỗi lần O(1). `mapIndexed` duyệt n ô, mỗi ô O(1). Hai bước nối tiếp nhau nên cộng lại, vẫn là O(n) (không có vòng lặp lồng nhau nên không phải O(n²)).
- **Độ phức tạp bộ nhớ: O(n)** cho danh sách `boxes` và danh sách `Piece` trả về.

### 2.4. `ListScreen`: màn hình danh sách

```kotlin
@Composable
fun ListScreen(onShowSpiral: (Int) -> Unit) {
    var n by remember { mutableIntStateOf(10) }
    val fibs = remember(n) { fibonacci(n) }

    Slider(
        value = n.toFloat(),
        onValueChange = { n = it.toInt() },
        valueRange = 3f..20f,
        steps = 16
    )

    LazyColumn(modifier = Modifier.weight(1f)) {
        itemsIndexed(fibs) { i, f ->
            val ratio = if (i == 0) "—" else "%.6f".format(f.toDouble() / fibs[i - 1])
            Card(Modifier.fillMaxWidth()) {
                Row {
                    Text("F(${i + 1}) = $f")
                    Text("tỉ số: $ratio")
                }
            }
        }
    }

    Button(onClick = { onShowSpiral(n) }) { Text("Xem xoắn ốc Fibonacci") }
}
```

(Đoạn trên là bản rút gọn, bỏ phần Scaffold, padding và định dạng; bản đầy đủ nằm trong `SourceCode/MainActivity.kt`.)

- **State `n`:** `remember { mutableIntStateOf(10) }` lưu N hiện tại. Khi `n` đổi, Compose vẽ lại màn hình (recomposition).
- **`remember(n)`:** chỉ tính lại dãy Fibonacci khi `n` thay đổi, không tính lại ở mỗi lần vẽ.
- **`Slider`:** khoảng 3 đến 20, `steps = 16` để chọn đúng các số nguyên (có 18 giá trị, tức 16 điểm dừng ở giữa).
- **`LazyColumn` + `itemsIndexed`:** chỉ tạo giao diện cho các dòng đang hiển thị, lấy được cả chỉ số `i` và giá trị `f`. Tỉ số là `f / fibs[i − 1]`; dòng đầu (`i == 0`) không có số trước nên hiện dấu "—".
- **`Button`:** gọi callback `onShowSpiral(n)` để màn hình cha điều hướng.
- **Độ phức tạp:** tính dãy O(n) (chỉ khi n đổi); mỗi dòng tính tỉ số O(1); tổng chi phí dựng danh sách là O(n). Vì `LazyColumn` chỉ dựng các dòng nhìn thấy nên thực tế còn ít hơn.

### 2.5. `SpiralScreen`: màn hình xoắn ốc (Canvas)

```kotlin
// Trích rút gọn: phần Canvas của SpiralScreen
val fibs = remember(n) { fibonacci(n) }
val pieces = remember(n) { buildPieces(fibs) }
val measurer = rememberTextMeasurer()

Canvas(Modifier.fillMaxSize()) {
    // Co giãn vừa màn hình
    val minX = pieces.minOf { it.l }
    val minY = pieces.minOf { it.t }
    val w = pieces.maxOf { it.r } - minX
    val h = pieces.maxOf { it.b } - minY
    val sc = min(size.width / w, size.height / h)
    val offX = (size.width - w * sc) / 2f - minX * sc
    val offY = (size.height - h * sc) / 2f - minY * sc
    fun sx(x: Float) = x * sc + offX
    fun sy(y: Float) = y * sc + offY

    pieces.forEachIndexed { i, p ->
        val color = palette[i % palette.size]
        val topLeft = Offset(sx(p.l), sy(p.t))
        val sz = Size(p.side * sc, p.side * sc)

        // Ô vuông: nền nhạt + viền
        drawRect(color.copy(alpha = 0.35f), topLeft, sz)
        drawRect(color, topLeft, sz, style = Stroke(width = 2f))

        // Ghi số khi ô đủ lớn
        if (p.side * sc > 28f) {
            drawText(measurer, "${fibs[i]}", topLeft = Offset(topLeft.x + 6f, topLeft.y + 4f))
        }

        // Cung 1/4 đường tròn
        val ox = if (p.cx == p.l) p.r else p.l
        val oy = if (p.cy == p.t) p.b else p.t
        val angA = Math.toDegrees(atan2(0f, ox - p.cx).toDouble()).toFloat()
        val angB = Math.toDegrees(atan2(oy - p.cy, 0f).toDouble()).toFloat()
        val sweep = ((angB - angA + 540f) % 360f) - 180f   // +90 hoặc -90 độ
        val rad = p.side * sc
        drawArc(
            color = Color(0xFFD81B60),
            startAngle = angA,
            sweepAngle = sweep,
            useCenter = false,
            topLeft = Offset(sx(p.cx) - rad, sy(p.cy) - rad),
            size = Size(rad * 2, rad * 2),
            style = Stroke(width = 6f, cap = StrokeCap.Round)
        )
    }
}
```

- **Dữ liệu:** `fibs` và `pieces` được `remember(n)`, tính một lần cho mỗi giá trị n.
- **Co giãn vừa màn hình:** tính khung bao toàn bộ ô (`minX`, `minY`, `w`, `h`), tỉ lệ phóng `sc = min(rộng canvas / w, cao canvas / h)`, rồi độ dời để căn giữa. Hai hàm `sx` và `sy` đổi toạ độ logic sang pixel.
- **Vẽ ô:** `drawRect` hai lần, một lần tô nền nhạt (alpha 0,35), một lần vẽ viền bằng `Stroke`.
- **Ghi số:** `drawText` với `TextMeasurer`, chỉ ghi khi cạnh ô trên màn hình lớn hơn 28 pixel để tránh chữ chồng nhau.
- **Vẽ cung:** tâm `(cx, cy)`, bán kính bằng cạnh ô. Hai đầu mút là hai góc kề với tâm. Góc bắt đầu tính bằng `atan2` rồi đổi sang độ. Góc quét `sweep = ((angB − angA + 540) % 360) − 180` luôn cho kết quả +90 hoặc −90 độ, tức cung ngắn một phần tư đường tròn. Trong Compose, góc 0 độ ở hướng 3 giờ và tăng theo chiều kim đồng hồ. `useCenter = false` để chỉ vẽ đường cong.
- **Độ phức tạp thời gian: O(n).** Tính khung bao gồm bốn lần duyệt (`minOf`, `maxOf`, mỗi lần O(n)), sau đó `forEachIndexed` duyệt n ô, mỗi ô thực hiện một số lệnh vẽ cố định O(1). Tổng O(n), không có vòng lặp lồng nhau.
- **Độ phức tạp bộ nhớ:** O(n) cho `fibs` và `pieces`; phần vẽ chỉ dùng O(1) bộ nhớ thêm.

### 2.6. `FibonacciApp`: điều hướng (Navigation)

```kotlin
@Composable
fun FibonacciApp() {
    val nav = rememberNavController()
    NavHost(navController = nav, startDestination = "list") {
        composable("list") {
            ListScreen(onShowSpiral = { n -> nav.navigate("spiral/$n") })
        }
        composable(
            route = "spiral/{n}",
            arguments = listOf(navArgument("n") { type = NavType.IntType })
        ) { entry ->
            val n = entry.arguments?.getInt("n") ?: 8
            SpiralScreen(n = n, onBack = { nav.popBackStack() })
        }
    }
}
```

- `rememberNavController()` tạo bộ điều khiển điều hướng, giữ ngăn xếp các màn hình.
- `NavHost` khai báo các màn hình; màn hình bắt đầu là `list`.
- Route `spiral/{n}` có tham số `n` kiểu số nguyên (`NavType.IntType`). Gọi `navigate("spiral/$n")` để truyền N, nhận bằng `entry.arguments?.getInt("n")`, mặc định 8 nếu thiếu.
- `popBackStack()` quay lại màn hình trước.
- **Độ phức tạp:** mỗi thao tác điều hướng là O(1).

### 2.7. `MainActivity`: điểm vào ứng dụng

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme { FibonacciApp() }
        }
    }
}
```

- Kế thừa `ComponentActivity`; trong `onCreate` gọi `setContent` để gắn giao diện Compose, bọc bằng `MaterialTheme` rồi gọi `FibonacciApp()`.

### 2.8. Tổng hợp độ phức tạp

| Thành phần | Thời gian | Bộ nhớ |
|---|---|---|
| `fibonacci(n)` | O(n) | O(n) |
| `buildPieces(fibs)` | O(n) | O(n) |
| `ListScreen` | O(n) | O(n) |
| `SpiralScreen` (Canvas) | O(n) | O(n) |
| Navigation | O(1) | O(1) |
| **Toàn chương trình** | **O(n)** | **O(n)** |

Chương trình không có hai vòng lặp lồng nhau nên không có thành phần O(n²). Với n ≤ 20, ứng dụng chạy gần như tức thời.

---

## 3. Điểm khác so với sách

- Sách dùng JavaFX để vẽ trên máy tính; bản này chạy trên Android bằng Jetpack Compose.
- Có thêm tương tác chọn N, điều hướng giữa hai màn hình, danh sách `LazyColumn` và tự co giãn hình vừa màn hình.

## 4. Hướng phát triển

- Thêm animation vẽ xoắn ốc từng bước.
- Cho nhập N bằng bàn phím và phóng to, thu nhỏ hình.
- Dùng `rememberSaveable` để giữ giá trị N khi xoay màn hình.
- Viết unit test cho `fibonacci()` và `buildPieces()`.

## 5. Nguồn tham khảo

- Sách *Kotlin from Scratch*, Part II, Chapter 4, Project 13.
