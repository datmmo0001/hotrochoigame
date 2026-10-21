mkdir -p app/src/main/res/xml app/src/main/res/drawable app/src/main/java/com/example/simplemacro

cat << 'EOF' > settings.gradle
pluginManagement { repositories { google(); mavenCentral(); gradlePluginPortal() } }
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories { google(); mavenCentral() }
}
rootProject.name = "SimpleMacro"
include ':app'
EOF

cat << 'EOF' > build.gradle
plugins {
    id 'com.android.application' version '8.2.2' apply false
    id 'org.jetbrains.kotlin.android' version '1.9.22' apply false
}
EOF

cat << 'EOF' > gradle.properties
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
android.useAndroidX=true
EOF

cat << 'EOF' > app/build.gradle
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
}
android {
    namespace 'com.example.simplemacro'
    compileSdk 34
    defaultConfig {
        applicationId "com.example.simplemacro"
        minSdk 24
        targetSdk 34
        versionCode 4
        versionName "4.0"
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }
    kotlinOptions { jvmTarget = '17' }
}
dependencies {
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'
}
EOF

cat << 'EOF' > app/src/main/res/drawable/ic_star_icon.xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    <path
        android:fillColor="#141824"
        android:pathData="M0,0h108v108h-108z"/>
    <path
        android:fillColor="#FFD700"
        android:strokeColor="#FFA000"
        android:strokeWidth="2"
        android:pathData="M54,14 L65.8,38.2 L92.5,42.0 L73.2,60.8 L77.8,87.4 L54,74.8 L30.2,87.4 L34.8,60.8 L15.5,42.0 L42.2,38.2 Z"/>
</vector>
EOF

cat << 'EOF' > app/src/main/AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
    <application
        android:label="Auto Click Pro"
        android:icon="@drawable/ic_star_icon"
        android:roundIcon="@drawable/ic_star_icon"
        android:theme="@style/Theme.AppCompat.NoActionBar">
        <activity android:name=".MainActivity" android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <service android:name=".MacroRecorderService" android:exported="true"
            android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService" />
            </intent-filter>
            <meta-data android:name="android.accessibilityservice" android:resource="@xml/accessibility_service_config" />
        </service>
    </application>
</manifest>
EOF

cat << 'EOF' > app/src/main/res/xml/accessibility_service_config.xml
<?xml version="1.0" encoding="utf-8"?>
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:accessibilityEventTypes="typeAllMask"
    android:accessibilityFeedbackType="feedbackGeneric"
    android:canPerformGestures="true"
    android:canRetrieveWindowContent="true" />
EOF

cat << 'EOF' > app/src/main/java/com/example/simplemacro/MainActivity.kt
package com.example.simplemacro
import android.content.Intent
import android.graphics.Color
import android.graphics.Typeface
import android.graphics.drawable.GradientDrawable
import android.net.Uri
import android.os.Bundle
import android.provider.Settings
import android.view.Gravity
import android.widget.*
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {
    override fun onCreate(s: Bundle?) {
        super.onCreate(s)
        val d = resources.displayMetrics.density
        val root = FrameLayout(this).apply { setBackgroundColor(Color.parseColor("#0F172A")) }

        val col = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            gravity = Gravity.CENTER_HORIZONTAL
            setPadding((24 * d).toInt(), (48 * d).toInt(), (24 * d).toInt(), (48 * d).toInt())
        }

        val starLogo = TextView(this).apply {
            text = "★"
            textSize = 54f
            setTextColor(Color.parseColor("#FFD700"))
            gravity = Gravity.CENTER
        }

        val title = TextView(this).apply {
            text = "AUTO CLICK PRO"
            textSize = 24f
            typeface = Typeface.DEFAULT_BOLD
            setTextColor(Color.WHITE)
            gravity = Gravity.CENTER
            setPadding(0, (8 * d).toInt(), 0, (4 * d).toInt())
        }

        val sub = TextView(this).apply {
            text = "Ghi chuỗi Click & Vuốt chính xác theo thời gian thực"
            textSize = 14f
            setTextColor(Color.parseColor("#94A3B8"))
            gravity = Gravity.CENTER
            setPadding(0, 0, 0, (28 * d).toInt())
        }

        val card = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            background = GradientDrawable().apply {
                setColor(Color.parseColor("#1E293B"))
                cornerRadius = 16 * d
                setStroke((1 * d).toInt(), Color.parseColor("#334155"))
            }
            setPadding((20 * d).toInt(), (24 * d).toInt(), (20 * d).toInt(), (24 * d).toInt())
        }

        val btn1 = mkMainBtn("1. Cấp quyền Hiển thị trên ứng dụng khác", "#3B82F6", d) {
            if (!Settings.canDrawOverlays(this))
                startActivity(Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:$packageName")))
            else Toast.makeText(this, "Đã cấp quyền Hiển thị!", Toast.LENGTH_SHORT).show()
        }

        val btn2 = mkMainBtn("2. Bật Hỗ trợ tiếp cận (Auto Click Pro)", "#10B981", d) {
            startActivity(Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS))
        }

        val tip = TextView(this).apply {
            text = "• Bước 1 được thực thi ngay khi bấm Chạy.\n• Các bước tiếp theo tự động chờ đúng số giây (2s, 5s...) như lúc bạn thao tác thật.\n• Vào mục 'Kịch bản' để lưu hoặc chỉnh sửa từng giây."
            textSize = 13.5f
            setTextColor(Color.parseColor("#CBD5E1"))
            setLineSpacing(6 * d, 1f)
            setPadding(0, (12 * d).toInt(), 0, 0)
        }

        card.apply { addView(btn1); addView(btn2); addView(tip) }
        col.apply { addView(starLogo); addView(title); addView(sub); addView(card) }

        val sig = TextView(this).apply {
            text = "made by Datnd88"
            textSize = 13f
            typeface = Typeface.create(Typeface.DEFAULT, Typeface.BOLD_ITALIC)
            setTextColor(Color.parseColor("#FFD700"))
            layoutParams = FrameLayout.LayoutParams(-2, -2).apply {
                gravity = Gravity.BOTTOM or Gravity.END
                setMargins(0, 0, (20 * d).toInt(), (16 * d).toInt())
            }
        }

        root.apply { addView(col); addView(sig) }
        setContentView(root)
    }

    private fun mkMainBtn(txt: String, hex: String, d: Float, clk: () -> Unit) = Button(this).apply {
        text = txt; textSize = 14.5f; isAllCaps = false; setTextColor(Color.WHITE)
        typeface = Typeface.DEFAULT_BOLD
        setPadding((16 * d).toInt(), (14 * d).toInt(), (16 * d).toInt(), (14 * d).toInt())
        background = GradientDrawable().apply { setColor(Color.parseColor(hex)); cornerRadius = 12 * d }
        layoutParams = LinearLayout.LayoutParams(-1, -2).apply { bottomMargin = (14 * d).toInt() }
        setOnClickListener { clk() }
    }
}
EOF
cat << 'EOF' > app/src/main/java/com/example/simplemacro/MacroRecorderService.kt
package com.example.simplemacro
import android.accessibilityservice.*
import android.app.AlertDialog
import android.graphics.*
import android.graphics.drawable.GradientDrawable
import android.os.*
import android.text.InputType
import android.view.*
import android.view.accessibility.AccessibilityEvent
import android.widget.*
import java.util.Locale
import kotlin.math.abs

data class Step(val sx: Float, val sy: Float, val ex: Float, val ey: Float, val dur: Long, var delay: Long)

class MacroRecorderService : AccessibilityService() {
    private lateinit var wm: WindowManager
    private lateinit var root: FrameLayout
    private lateinit var panel: LinearLayout
    private lateinit var bubble: TextView
    private lateinit var tvStatus: TextView
    private lateinit var params: WindowManager.LayoutParams
    private lateinit var bRec: Button
    private lateinit var bPlay: Button
    private lateinit var bLoop: Button
    private var overlay: View? = null
    private val steps = mutableListOf<Step>()
    private var isRec = false
    private var isPlay = false
    private var lastUpTime = 0L
    private val h = Handler(Looper.getMainLooper())
    private var loopMode = 0
    private var maxLoop = 5
    private var curLoop = 0
    private var dx = 0f; private var dy = 0f; private var dt = 0L

    override fun onServiceConnected() {
        super.onServiceConnected()
        wm = getSystemService(WINDOW_SERVICE) as WindowManager
        initUI()
    }

    private fun initUI() {
        val d = resources.displayMetrics.density
        root = FrameLayout(this)

        panel = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            background = GradientDrawable().apply {
                setColor(Color.parseColor("#F2111827"))
                cornerRadius = 18 * d
                setStroke((1.5f * d).toInt(), Color.parseColor("#FFD700"))
            }
            setPadding((14 * d).toInt(), (10 * d).toInt(), (14 * d).toInt(), (8 * d).toInt())
        }

        // Hàng 1: Thanh tiêu đề kéo thả + Trạng thái + Thu nhỏ + Thoát
        val topRow = LinearLayout(this).apply {
            orientation = LinearLayout.HORIZONTAL
            gravity = Gravity.CENTER_VERTICAL
            layoutParams = LinearLayout.LayoutParams(-1, -2).apply { bottomMargin = (8 * d).toInt() }
        }

        val dragTitle = TextView(this).apply {
            text = "✥  ★ AUTO PRO"
            textSize = 13.5f
            typeface = Typeface.DEFAULT_BOLD
            setTextColor(Color.parseColor("#FFD700"))
            setPadding((4 * d).toInt(), (4 * d).toInt(), (10 * d).toInt(), (4 * d).toInt())
        }
        makeDraggable(dragTitle, null)

        tvStatus = TextView(this).apply {
            text = "Sẵn sàng (0 bước)"
            textSize = 12f
            setTextColor(Color.parseColor("#9CA3AF"))
            layoutParams = LinearLayout.LayoutParams(0, -2, 1f)
        }
        makeDraggable(tvStatus, null)

        val bMin = mkSmallBtn("—", "#374151", d) { setBubble(true) }
        val bExit = mkSmallBtn("✕", "#991B1B", d) { exitApp() }
        topRow.apply { addView(dragTitle); addView(tvStatus); addView(bMin); addView(bExit) }

        // Hàng 2: Các nút chức năng chính to rõ
        val btnRow = LinearLayout(this).apply {
            orientation = LinearLayout.HORIZONTAL
            gravity = Gravity.CENTER_VERTICAL
        }

        bRec = mkActionBtn("● Ghi", "#EF4444", d) { if (!isPlay) if (isRec) stopRec() else startRec() }
        bPlay = mkActionBtn("▶ Chạy", "#10B981", d) {
            if (!isRec) if (isPlay) stopPlay() else if (steps.isNotEmpty()) startPlay() else msg("Hãy bấm 'Ghi' hoặc mở 'Kịch bản' trước!")
        }
        bLoop = mkActionBtn("↻ 1 Lần", "#3B82F6", d) {
            loopMode = (loopMode + 1) % 3
            when (loopMode) {
                0 -> bLoop.text = "↻ 1 Lần"
                1 -> { bLoop.text = "↻ $maxLoop Lần"; askLoop() }
                2 -> bLoop.text = "↻ Vô hạn"
            }
        }
        val bFile = mkActionBtn("📁 Kịch bản", "#D97706", d) { if (!isRec && !isPlay) showMacroMenu() }
        btnRow.apply { addView(bRec); addView(bPlay); addView(bLoop); addView(bFile) }

        // Hàng 3: Chữ ký góc dưới cùng bên phải
        val sig = TextView(this).apply {
            text = "made by Datnd88"
            textSize = 10.5f
            typeface = Typeface.create(Typeface.DEFAULT, Typeface.ITALIC)
            setTextColor(Color.parseColor("#FBBF24"))
            gravity = Gravity.END
            layoutParams = LinearLayout.LayoutParams(-1, -2).apply { topMargin = (5 * d).toInt() }
        }

        panel.apply { addView(topRow); addView(btnRow); addView(sig) }

        // Bong bóng chat hình Ngôi sao vàng
        bubble = TextView(this).apply {
            text = "★\nAuto"
            setTextColor(Color.parseColor("#111827"))
            typeface = Typeface.DEFAULT_BOLD
            textSize = 12f
            gravity = Gravity.CENTER
            visibility = View.GONE
            background = GradientDrawable().apply {
                shape = GradientDrawable.OVAL
                setColor(Color.parseColor("#FFD700"))
                setStroke((2.5f * d).toInt(), Color.WHITE)
            }
            val s = (56 * d).toInt()
            layoutParams = FrameLayout.LayoutParams(s, s)
        }
        makeDraggable(bubble) { if (isPlay) stopPlay(); setBubble(false) }

        root.apply { addView(panel); addView(bubble) }
        params = WindowManager.LayoutParams(-2, -2, WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY, 8, -3).apply {
            gravity = Gravity.TOP or Gravity.START; x = 30; y = 140
        }
        wm.addView(root, params)
    }

    private fun mkActionBtn(txt: String, col: String, d: Float, clk: () -> Unit) = Button(this).apply {
        text = txt; textSize = 13f; isAllCaps = false; setTextColor(Color.WHITE)
        typeface = Typeface.DEFAULT_BOLD
        minWidth = 0; minimumWidth = 0; minHeight = 0; minimumHeight = 0
        setPadding((12 * d).toInt(), (9 * d).toInt(), (12 * d).toInt(), (9 * d).toInt())
        background = GradientDrawable().apply { setColor(Color.parseColor(col)); cornerRadius = 10 * d }
        layoutParams = LinearLayout.LayoutParams(-2, -2).apply { marginEnd = (6 * d).toInt() }
        setOnClickListener { clk() }
    }

    private fun mkSmallBtn(txt: String, col: String, d: Float, clk: () -> Unit) = Button(this).apply {
        text = txt; textSize = 12f; isAllCaps = false; setTextColor(Color.WHITE)
        typeface = Typeface.DEFAULT_BOLD
        minWidth = 0; minimumWidth = 0; minHeight = 0; minimumHeight = 0
        setPadding((10 * d).toInt(), (4 * d).toInt(), (10 * d).toInt(), (4 * d).toInt())
        background = GradientDrawable().apply { setColor(Color.parseColor(col)); cornerRadius = 8 * d }
        layoutParams = LinearLayout.LayoutParams(-2, -2).apply { marginStart = (6 * d).toInt() }
        setOnClickListener { clk() }
    }
EOF
cat << 'EOF' >> app/src/main/java/com/example/simplemacro/MacroRecorderService.kt

    private fun exitApp() {
        stopPlay(); hideOverlay()
        if (::root.isInitialized && root.isAttachedToWindow) wm.removeView(root)
        msg("Đã tắt Auto Click Pro!")
        disableSelf()
    }

    private fun updateStatus() {
        tvStatus.text = if (steps.isEmpty()) "Sẵn sàng (0 bước)" else "Đã nạp: ${steps.size} bước"
    }

    private fun showMacroMenu() {
        val prefs = getSharedPreferences("macros", MODE_PRIVATE)
        val names = prefs.all.keys.toTypedArray()
        val opts = mutableListOf(
            "➕ Lưu kịch bản hiện tại (${steps.size} bước)",
            "⏱️ Xem & Chỉnh số giây chờ từng bước",
            "🗑️ Xóa trắng kịch bản hiện tại"
        )
        names.forEach { opts.add("▶️ Tải kịch bản: $it") }
        names.forEach { opts.add("❌ Xóa vĩnh viễn: $it") }

        AlertDialog.Builder(this, android.R.style.Theme_DeviceDefault_Dialog_Alert)
            .setTitle("★ Quản lý Kịch bản")
            .setItems(opts.toTypedArray()) { _, w ->
                when {
                    w == 0 -> saveMacroDialog()
                    w == 1 -> showStepTimingEditor()
                    w == 2 -> { steps.clear(); updateStatus(); msg("Đã xóa bộ nhớ tạm!") }
                    w - 3 < names.size -> {
                        val name = names[w - 3]
                        loadMacro(prefs.getString(name, "") ?: "", name)
                    }
                    else -> {
                        val name = names[w - 3 - names.size]
                        prefs.edit().remove(name).apply()
                        msg("Đã xóa '$name'")
                    }
                }
            }.create().apply { window?.setType(WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY); show() }
    }

    private fun showStepTimingEditor() {
        if (steps.isEmpty()) { msg("Chưa có bước nào! Hãy bấm Ghi trước."); return }
        val items = steps.mapIndexed { i, s ->
            val sec = String.format(Locale.US, "%.1f", s.delay / 1000f)
            val type = if (abs(s.ex - s.sx) > 15f || abs(s.ey - s.sy) > 15f) "Vuốt" else "Click"
            if (i == 0) "Bước 1 ($type): Chạy ngay sau ${sec}s"
            else "Bước ${i + 1} ($type): Chờ ${sec}s sau Bước $i"
        }.toTypedArray()

        AlertDialog.Builder(this, android.R.style.Theme_DeviceDefault_Dialog_Alert)
            .setTitle("Chạm vào bước để sửa số giây chờ")
            .setItems(items) { _, idx -> editSingleStepDelay(idx) }
            .setPositiveButton("Đóng", null)
            .create().apply { window?.setType(WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY); show() }
    }

    private fun editSingleStepDelay(idx: Int) {
        val inp = EditText(this).apply {
            inputType = InputType.TYPE_CLASS_NUMBER or InputType.TYPE_NUMBER_FLAG_DECIMAL
            setText(String.format(Locale.US, "%.1f", steps[idx].delay / 1000f))
        }
        AlertDialog.Builder(this, android.R.style.Theme_DeviceDefault_Dialog_Alert)
            .setTitle("Số giây chờ trước Bước ${idx + 1}")
            .setView(inp)
            .setPositiveButton("Lưu") { _, _ ->
                inp.text.toString().toFloatOrNull()?.let { sec ->
                    steps[idx].delay = (sec * 1000).toLong().coerceAtLeast(50L)
                    msg("Đã đặt Bước ${idx + 1} chờ ${sec}s!")
                }
            }.setNegativeButton("Hủy", null)
            .create().apply { window?.setType(WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY); show() }
    }

    private fun saveMacroDialog() {
        if (steps.isEmpty()) { msg("Chưa có thao tác nào để lưu!"); return }
        val inp = EditText(this).apply { hint = "Nhập tên kịch bản..." }
        AlertDialog.Builder(this, android.R.style.Theme_DeviceDefault_Dialog_Alert)
            .setTitle("Lưu kịch bản").setView(inp)
            .setPositiveButton("Lưu") { _, _ ->
                val name = inp.text.toString().trim().ifEmpty { "Kịch bản ${System.currentTimeMillis() % 1000}" }
                val data = steps.joinToString(";") { "${it.sx},${it.sy},${it.ex},${it.ey},${it.dur},${it.delay}" }
                getSharedPreferences("macros", MODE_PRIVATE).edit().putString(name, data).apply()
                msg("Đã lưu '$name'!")
            }.setNegativeButton("Hủy", null).create().apply { window?.setType(WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY); show() }
    }

    private fun loadMacro(data: String, name: String) {
        if (data.isEmpty()) return
        steps.clear()
        data.split(";").forEach { item ->
            val p = item.split(",")
            if (p.size == 6) steps.add(Step(p[0].toFloat(), p[1].toFloat(), p[2].toFloat(), p[3].toFloat(), p[4].toLong(), p[5].toLong()))
        }
        updateStatus()
        msg("Đã tải '$name' (${steps.size} bước)")
    }

    private fun setBubble(b: Boolean) {
        panel.visibility = if (b) View.GONE else View.VISIBLE
        bubble.visibility = if (b) View.VISIBLE else View.GONE
        bubble.text = if (isPlay) "■\nDừng" else "★\nAuto"
        wm.updateViewLayout(root, params)
    }

    private fun makeDraggable(v: View, onClick: (() -> Unit)?) {
        var ix = 0; var iy = 0; var tx = 0f; var ty = 0f
        v.setOnTouchListener { _, e ->
            when (e.action) {
                0 -> { ix = params.x; iy = params.y; tx = e.rawX; ty = e.rawY; true }
                2 -> { params.x = ix + (e.rawX - tx).toInt(); params.y = iy + (e.rawY - ty).toInt(); wm.updateViewLayout(root, params); true }
                1 -> { if (abs(e.rawX - tx) < 15 && abs(e.rawY - ty) < 15) onClick?.invoke(); true }
                else -> false
            }
        }
    }

    private fun askLoop() {
        val inp = EditText(this).apply { inputType = InputType.TYPE_CLASS_NUMBER; setText("$maxLoop") }
        AlertDialog.Builder(this, android.R.style.Theme_DeviceDefault_Dialog_Alert)
            .setTitle("Nhập số vòng lặp").setView(inp)
            .setPositiveButton("Lưu") { _, _ -> inp.text.toString().toIntOrNull()?.let { if (it > 0) { maxLoop = it; bLoop.text = "↻ $it Lần" } } }
            .setNegativeButton("Hủy", null).create().apply { window?.setType(WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY); show() }
    }

    private fun startRec() {
        steps.clear(); isRec = true; bRec.text = "■ Dừng"
        tvStatus.text = "Đang ghi thao tác..."
        showOverlay(); msg("Hãy chạm/vuốt theo đúng nhịp độ bạn muốn!")
    }

    private fun stopRec() {
        isRec = false; bRec.text = "● Ghi"; hideOverlay(); updateStatus()
        msg("Đã lưu ${steps.size} bước chuẩn thời gian!")
    }

    private fun showOverlay() {
        if (overlay != null) return
        overlay = View(this).apply {
            setBackgroundColor(Color.parseColor("#1AFFD700"))
            setOnTouchListener { _, e ->
                if (!isRec) return@setOnTouchListener false
                if (e.action == 0) {
                    dx = e.rawX; dy = e.rawY; dt = System.currentTimeMillis()
                } else if (e.action == 1) {
                    val upNow = System.currentTimeMillis()
                    val dur = (upNow - dt).coerceIn(50L, 60000L)
                    // Bước 1 chạy ngay (100ms), các bước sau tính đúng khoảng nghỉ từ lúc nhấc tay bước trước
                    val del = if (steps.isEmpty()) 100L else (dt - lastUpTime).coerceAtLeast(80L)
                    lastUpTime = upNow

                    val s = Step(dx, dy, e.rawX, e.rawY, dur, del)
                    steps.add(s)
                    tvStatus.text = "Đã ghi: ${steps.size} bước (${String.format(Locale.US, "%.1fs", del / 1000f)})"
                    hideOverlay()
                    h.postDelayed({
                        doGesture(s) { if (isRec) h.postDelayed({ showOverlay() }, 80) }
                    }, 30)
                }
                true
            }
        }
        wm.addView(overlay, WindowManager.LayoutParams(-1, -1, WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY, 8, -3))
        wm.removeView(root); wm.addView(root, params)
    }

    private fun hideOverlay() { overlay?.let { if (it.isAttachedToWindow) wm.removeView(it); overlay = null } }
    private fun startPlay() { isPlay = true; curLoop = 0; bPlay.text = "■ Dừng"; setBubble(true); runStep(0) }
    private fun stopPlay() { isPlay = false; h.removeCallbacksAndMessages(null); bPlay.text = "▶ Chạy"; bubble.text = "★\nAuto"; updateStatus() }

    private fun runStep(i: Int) {
        if (!isPlay) return
        if (i >= steps.size) {
            curLoop++
            if ((loopMode == 1 && curLoop < maxLoop) || loopMode == 2) runStep(0) else { stopPlay(); msg("Đã hoàn thành kịch bản!") }
            return
        }
        val s = steps[i]
        h.postDelayed({ if (isPlay) doGesture(s) { runStep(i + 1) } }, s.delay)
    }

    private fun doGesture(s: Step, done: () -> Unit) {
        val p = Path().apply { moveTo(s.sx, s.sy); if (abs(s.ex - s.sx) > 15f || abs(s.ey - s.sy) > 15f) lineTo(s.ex, s.ey) }
        val g = GestureDescription.Builder().addStroke(GestureDescription.StrokeDescription(p, 0, s.dur)).build()
        dispatchGesture(g, object : GestureResultCallback() {
            override fun onCompleted(d: GestureDescription?) = done()
            override fun onCancelled(d: GestureDescription?) = done()
        }, null)
    }

    private fun msg(t: String) = Toast.makeText(this, t, Toast.LENGTH_SHORT).show()
    override fun onAccessibilityEvent(e: AccessibilityEvent?) {}
    override fun onInterrupt() {}
    override fun onDestroy() { super.onDestroy(); hideOverlay(); if (::root.isInitialized && root.isAttachedToWindow) wm.removeView(root) }
}
EOF
