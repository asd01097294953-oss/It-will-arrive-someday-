package com.natural.seupjjo

import android.app.Activity
import android.content.Intent
import android.graphics.*
import android.net.Uri
import android.os.Bundle
import android.view.MotionEvent
import android.view.View
import kotlin.math.*
import kotlin.random.Random

class MainActivity : Activity() {
    private lateinit var game: GameView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        game = GameView(this) { pickPet() }
        setContentView(game)
    }

    private fun pickPet() {
        val i = Intent(Intent.ACTION_OPEN_DOCUMENT).apply {
            addCategory(Intent.CATEGORY_OPENABLE)
            type = "image/*"
        }
        startActivityForResult(i, 77)
    }

    @Deprecated("Deprecated Android callback")
    override fun onActivityResult(
        requestCode: Int,
        resultCode: Int,
        data: Intent?
    ) {
        super.onActivityResult(requestCode, resultCode, data)

        if (requestCode == 77 && resultCode == RESULT_OK) {
            data?.data?.let { uri ->
                try {
                    contentResolver.takePersistableUriPermission(
                        uri,
                        Intent.FLAG_GRANT_READ_URI_PERMISSION
                    )
                } catch (_: Exception) {}

                game.setPet(uri)
            }
        }
    }
}

private data class Platform(
    var x: Float,
    var y: Float,
    var width: Float,
    var kind: Int,
    var baseX: Float,
    var phase: Float
)

private data class Coin(
    val x: Float,
    val y: Float,
    var got: Boolean = false
)

private class GameView(
    private val activity: Activity,
    val choosePet: () -> Unit
) : View(activity) {

    private val prefs = activity.getSharedPreferences("natural_save", 0)
    private val paint = Paint(Paint.ANTI_ALIAS_FLAG)
    private val random = Random(20261105)

    private val zones = intArrayOf(0, 200, 400, 600, 800, 1000)

    private val zoneNames = arrayOf(
        "초원", "구름", "설산",
        "수정 동굴", "하늘 성", "정상"
    )

    private val abilityNames = arrayOf(
        "점프 강화",
        "2단 점프",
        "낙하 조절",
        "이동 강화",
        "공중 조작",
        "보호 부적",
        "발판 탐지",
        "코인 보너스",
        "착지 보호",
        "부활 깃털"
    )

    private val levels = IntArray(10) {
        prefs.getInt("ability_$it", 0)
    }

    private var state = 0

    private var heightM = prefs.getInt("height", 0).toFloat()
    private var best = prefs.getInt("best", 0)
    private var checkpoint = prefs.getInt("checkpoint", 0)
    private var coins = prefs.getInt("coins", 0)

    private var px = .5f
    private var py = checkpoint * 3f
    private var vy = 0f
    private var vx = 0f

    private var cam = 0f
    private var left = false
    private var right = false
    private var jumpHeld = false

    private var jumpUsed = 0
    private var last = System.nanoTime()

    private var toastText = ""
    private var toastUntil = 0L

    private var petUri: Uri? =
        prefs.getString("pet_uri", null)?.let { Uri.parse(it) }

    private var pet: Bitmap? = null

    private val mainBg =
        BitmapFactory.decodeResource(resources, R.drawable.main_bg)

    private val clearBg =
        BitmapFactory.decodeResource(resources, R.drawable.clear_bg)

    private val character =
        BitmapFactory.decodeResource(resources, R.drawable.character)

    private val platforms = mutableListOf<Platform>()
    private val coinList = mutableListOf<Coin>()

    private val coinCounts =
        intArrayOf(100, 120, 140, 160, 180, 100)

    init {
        isFocusable = true
        generateWorld()
        petUri?.let { loadPet(it) }
    }

    fun setPet(uri: Uri) {
        petUri = uri
        prefs.edit().putString("pet_uri", uri.toString()).apply()
        loadPet(uri)
        invalidate()
    }

    private fun loadPet(uri: Uri) {
        try {
            pet = activity.contentResolver
                .openInputStream(uri)
                ?.use { BitmapFactory.decodeStream(it) }
        } catch (_: Exception) {
            pet = null
        }
    }

    private fun save() {
        prefs.edit()
            .putInt("height", heightM.toInt())
            .putInt("best", best)
            .putInt("checkpoint", checkpoint)
            .putInt("coins", coins)
            .apply()

        levels.forEachIndexed { i, v ->
            prefs.edit().putInt("ability_$i", v).apply()
        }
    }

    private fun world(m: Float) = m * 3f

    private fun zone(m: Float) = when {
        m < 200 -> 0
        m < 400 -> 1
        m < 600 -> 2
        m < 800 -> 3
        m < 1000 -> 4
        else -> 5
    }

    private fun generateWorld() {
        platforms.clear()
        coinList.clear()

        platforms.add(
            Platform(.34f, 0f, .32f, 0, .34f, 0f)
        )

        for (z in 0..5) {
            val start = zones[z].toFloat()
            val end = if (z == 5) 1111f else start + 200f
            var m = start + 26f

            while (m < end - 8) {
                val x = random.nextFloat() * .76f + .04f

                val kind = when (random.nextInt(10)) {
                    0, 1 -> 1
                    2, 3 -> 2
                    else -> 0
                }

                platforms.add(
                    Platform(
                        x,
                        world(m),
                        .18f + random.nextFloat() * .13f,
                        kind,
                        x,
                        random.nextFloat() * 6.28f
                    )
                )

                m += 24f + random.nextFloat() * 20f
            }

            repeat(coinCounts[z]) {
                val pl = platforms
                    .filter {
                        it.y >= world(start) &&
                        it.y <= world(end)
                    }
                    .randomOrNull(random)

                if (pl != null) {
                    coinList.add(
                        Coin(
                            (pl.x + random.nextFloat() * pl.width)
                                .coerceIn(.02f, .98f),
                            pl.y + 17f
                        )
                    )
                }
            }

            if (z > 0) {
                platforms.add(
                    Platform(.06f, world(start), .88f, 0, .06f, 0f)
                )
            }
        }

        platforms.add(
            Platform(.05f, world(1111f), .90f, 0, .05f, 0f)
        )
    }

    private fun colorZone(m: Float) = when (zone(m)) {
        0 -> Color.rgb(139, 190, 91)
        1 -> Color.rgb(156, 205, 230)
        2 -> Color.rgb(226, 238, 244)
        3 -> Color.rgb(113, 83, 159)
        4 -> Color.rgb(198, 165, 83)
        else -> Color.rgb(245, 226, 150)
    }

    private fun txt(
        c: Canvas,
        s: String,
        x: Float,
        y: Float,
        size: Float,
        color: Int = Color.WHITE,
        center: Boolean = false
    ) {
        paint.color = color
        paint.textSize = size
        paint.typeface = Typeface.create(
            "sans-serif-rounded",
            Typeface.BOLD
        )
        paint.textAlign =
            if (center) Paint.Align.CENTER else Paint.Align.LEFT

        c.drawText(s, x, y, paint)
    }

    private fun rounded(
        c: Canvas,
        l: Float,
        t: Float,
        r: Float,
        b: Float,
        color: Int,
        rad: Float = 12f
    ) {
        paint.color = color
        c.drawRoundRect(l, t, r, b, rad, rad, paint)
    }

    private fun woodButton(
        c: Canvas,
        label: String,
        x: Float,
        y: Float,
        w: Float,
        h: Float
    ) {
        rounded(c, x + 3, y + 5, x + w + 3, y + h + 5,
            Color.rgb(55, 37, 22), 12f)

        rounded(c, x, y, x + w, y + h,
            Color.rgb(111, 70, 36), 12f)

        rounded(c, x + 5, y + 5, x + w - 5, y + h - 5,
            Color.rgb(158, 111, 62), 8f)

        txt(c, label, x + w / 2, y + h * .65f,
            h * .34f, Color.WHITE, true)
    }

    private fun imageBg(
        c: Canvas,
        b: Bitmap,
        w: Float,
        h: Float
    ) {
        paint.isFilterBitmap = true
        c.drawBitmap(b, null, RectF(0f, 0f, w, h), paint)
    }

    override fun onDraw(c: Canvas) {
        super.onDraw(c)

        val w = width.toFloat()
        val h = height.toFloat()

        when (state) {
            0 -> {
                imageBg(c, mainBg, w, h)

                txt(c, "자연스럽죠",
                    w * .10f, h * .15f, 32f, Color.WHITE)

                woodButton(c, "시작",
                    w * .08f, h * .58f, w * .40f, h * .095f)

                woodButton(c, "처음부터",
                    w * .27f, h * .72f, w * .48f, h * .095f)
            }

            1 -> {
                imageBg(c, mainBg, w, h)

                txt(c, "체크포인트 선택",
                    w / 2, h * .13f, 25f, Color.WHITE, true)

                for (i in 0..5) {
                    val x = if (i % 2 == 0) w * .08f else w * .53f
                    val y = h * .23f + (i / 2) * h * .19f
                    val cp = zones[i]

                    val unlocked = cp <= best || cp <= checkpoint

                    rounded(
                        c, x, y, x + w * .39f, y + h * .13f,
                        if (unlocked) Color.rgb(103, 135, 72)
                        else Color.argb(180, 45, 45, 45),
                        12f
                    )

                    txt(
                        c,
                        if (unlocked) "${cp}m ${zoneNames[i]}" else "🔒",
                        x + w * .195f,
                        y + h * .08f,
                        16f,
                        Color.WHITE,
                        true
                    )
                }

                woodButton(c, "뒤로",
                    w * .32f, h * .87f, w * .36f, h * .075f)
            }

            2 -> drawGame(c, w, h)

            3, 4 -> {
                drawGame(c, w, h)

                rounded(
                    c,
                    w * .06f, h * .19f,
                    w * .94f, h * .82f,
                    Color.argb(238, 34, 47, 34),
                    20f
                )

                if (state == 3) {
                    txt(c, "일시정지",
                        w / 2, h * .30f, 27f, Color.WHITE, true)

                    woodButton(c, "리셋",
                        w * .25f, h * .40f, w * .50f, h * .085f)

                    woodButton(c, "나가기",
                        w * .25f, h * .52f, w * .50f, h * .085f)

                    woodButton(c, "펫 사진 선택",
                        w * .18f, h * .64f, w * .64f, h * .08f)
                } else {
                    txt(c, "자연 상점",
                        w / 2, h * .27f, 26f, Color.WHITE, true)

                    txt(c, "보유 코인: $coins",
                        w / 2, h * .32f, 17f, Color.YELLOW, true)

                    for (i in abilityNames.indices) {
                        val col = i % 2
                        val row = i / 2
                        val x = w * .10f + col * w * .42f
                        val y = h * .39f + row * h * .065f
                        val lv = levels[i]

                        txt(c, "${abilityNames[i]} Lv.$lv",
                            x, y, 13f)

                        txt(c,
                            if (lv >= 5) "MAX" else "20코인",
                            x, y + 16, 11f, Color.YELLOW)
                    }

                    woodButton(c, "닫기",
                        w * .32f, h * .79f, w * .36f, h * .07f)
                }
            }

            5 -> {
                imageBg(c, clearBg, w, h)

                woodButton(c, "메인 화면으로",
                    w * .20f, h * .84f, w * .60f, h * .09f)
            }
        }
    }

    private fun drawGame(c: Canvas, w: Float, h: Float) {
        val bg = colorZone(heightM)
        c.drawColor(bg)

        val floorY = h * .72f
        cam = py - floorY

        paint.color = Color.argb(35, 255, 255, 255)

        for (i in 0..7) {
            c.drawCircle(
                (i * 137 % w.toInt()).toFloat(),
                (i * 83 % h.toInt()).toFloat(),
                18f,
                paint
            )
        }

        platforms.forEach { pl ->
            val sy = floorY - (pl.y - cam)

            if (sy in -30f..h + 30f) {
                if (pl.kind == 2) {
                    pl.x = pl.baseX +
                        sin((System.nanoTime() / 1e9 + pl.phase).toFloat()) * .045f
                }

                val col = when (zone(pl.y / 3f)) {
                    0 -> Color.rgb(78, 115, 48)
                    1 -> Color.rgb(231, 244, 250)
                    2 -> Color.rgb(180, 205, 216)
                    3 -> Color.rgb(135, 100, 190)
                    4 -> Color.rgb(174, 139, 65)
                    else -> Color.rgb(230, 209, 135)
                }

                rounded(
                    c, pl.x * w, sy,
                    (pl.x + pl.width) * w, sy + 13f,
                    col, 5f
                )

                if (pl.kind == 1) {
                    txt(c, "↑",
                        (pl.x + pl.width / 2) * w,
                        sy - 2, 15f, Color.WHITE, true)
                }
            }
        }

        coinList.filter { !it.got }.forEach { co ->
            val sy = floorY - (co.y - cam)

            if (sy in -20f..h + 20f) {
                paint.color = Color.rgb(255, 204, 35)
                c.drawCircle(co.x * w, sy, 6f, paint)

                paint.color = Color.rgb(255, 244, 150)
                c.drawCircle(co.x * w - 2, sy - 2, 2f, paint)
            }
        }

        val cx = px * w
        val cy = floorY
        val cw = 66f
        val ch = 86f

        paint.isFilterBitmap = false

        c.drawBitmap(
            character,
            null,
            RectF(cx - cw / 2, cy - ch + 5, cx + cw / 2, cy + 5),
            paint
        )

        pet?.let {
            val size = 30f
            paint.isFilterBitmap = true

            c.drawBitmap(
                it,
                null,
                RectF(cx + 17, cy - ch - 24,
                    cx + 17 + size, cy - ch + 6),
                paint
            )
        }

        txt(c, "${heightM.toInt()}m", 16f, 31f, 20f)
        txt(c, "$coins", w - 18f, 31f, 19f, Color.YELLOW)

        if (checkpoint > 0 && abs(heightM - checkpoint) < 8f) {
            rounded(c, w * .35f, h * .10f,
                w * .65f, h * .16f,
                Color.rgb(95, 67, 38), 10f)

            txt(c, "상점",
                w * .50f, h * .14f, 15f, Color.WHITE, true)
        }

        rounded(c, 14f, h - 88f, 82f, h - 20f,
            Color.argb(135, 35, 35, 35), 16f)

        txt(c, "◀", 48f, h - 43f, 26f, Color.WHITE, true)

        rounded(c, 90f, h - 88f, 158f, h - 20f,
            Color.argb(135, 35, 35, 35), 16f)

        txt(c, "▶", 124f, h - 43f, 26f, Color.WHITE, true)

        rounded(c, w - 94f, h - 99f, w - 17f, h - 18f,
            Color.argb(150, 35, 35, 35), 18f)

        txt(c, "점프",
            w - 55f, h - 51f, 17f, Color.WHITE, true)

        rounded(c, w - 54f, 8f, w - 10f, 52f,
            Color.argb(150, 35, 35, 35), 10f)

        txt(c, "Ⅱ", w - 32f, 37f, 20f, Color.WHITE, true)

        if (toastText.isNotEmpty() &&
            System.currentTimeMillis() < toastUntil
        ) {
            txt(c, toastText,
                w / 2, h * .22f, 20f, Color.WHITE, true)
        }
    }

    private fun startAt(cp: Int) {
        checkpoint = max(checkpoint, cp)
        heightM = cp.toFloat()
        py = world(cp.toFloat())
        px = .5f
        vy = 0f
        vx = 0f
        jumpUsed = 0
        state = 2
        save()
    }

    private fun update(dt: Float) {
        if (state != 2) return

        val moveSpeed = .62f + levels[3] * .08f
        val accel = if (levels[4] > 0) 1.7f else 1f

        if (left) px -= moveSpeed * dt
        if (right) px += moveSpeed * dt

        px = px.coerceIn(.035f, .965f)

        if (jumpHeld && vy >= -9f) {
            vy -= .045f * dt * 60f *
                (1f + levels[0] * .17f)
        }

        vy += .055f * dt * 60f
        vy = vy.coerceIn(-13f, 13f)

        val old = py
        py += vy * dt * 60f

        if (vy > 0) {
            val hit = platforms.firstOrNull { pl ->
                old <= pl.y + 5f &&
                py >= pl.y &&
                px >= pl.x - .035f &&
                px <= pl.x + pl.width + .035f
            }

            if (hit != null) {
                py = hit.y
                vy = 0f
                jumpUsed = 0

                if (hit.kind == 1) {
                    vy = -(8f + levels[0] * .7f)
                }
            }
        }

        if (py < world(checkpoint.toFloat())) {
            py = world(checkpoint.toFloat())
            vy = 0f
            heightM = checkpoint.toFloat()
            jumpUsed = 0
        }

        val current = py / 3f

        if (current > heightM) {
            heightM = current
            best = max(best, heightM.toInt())
        }

        coinList.forEach { co ->
            if (!co.got &&
                abs(py - co.y) < 24f &&
                abs(px - co.x) < .065f
            ) {
                co.got = true
                coins += 1 + levels[7]
                toastText = "+${1 + levels[7]}"
                toastUntil = System.currentTimeMillis() + 700
            }
        }

        zones.drop(1).forEach { cp ->
            if (heightM >= cp && checkpoint < cp) {
                checkpoint = cp
                save()
                toastText = "${cp}m 도달!"
                toastUntil = System.currentTimeMillis() + 1600
            }
        }

        if (heightM >= 1111f) {
            heightM = 1111f
            best = 1111
            state = 5
            save()
        }

        if (heightM.toInt() % 25 == 0) save()
    }

    override fun onAttachedToWindow() {
        super.onAttachedToWindow()

        post(object : Runnable {
            override fun run() {
                val now = System.nanoTime()

                val dt = ((now - last) / 1e9)
                    .toFloat()
                    .coerceIn(0f, .04f)

                last = now
                update(dt)
                invalidate()

                if (isAttachedToWindow) {
                    postDelayed(this, 16)
                }
            }
        })
    }

    override fun onTouchEvent(e: MotionEvent): Boolean {
        val x = e.x
        val y = e.y
        val w = width.toFloat()
        val h = height.toFloat()

        when (e.action) {
            MotionEvent.ACTION_DOWN -> {
                when (state) {
                    0 -> {
                        if (y in h * .56f..h * .69f) {
                            state = 1
                        } else if (y >= h * .69f) {
                            prefs.edit().clear().apply()
                            levels.fill(0)
                            coins = 0
                            best = 0
                            checkpoint = 0
                            heightM = 0f
                            generateWorld()
                            save()
                            state = 1
                        }
                    }

                    1 -> {
                        if (y > h * .86f) {
                            state = 0
                        } else {
                            val col = if (x > w * .5f) 1 else 0
                            val row = ((y - h * .23f) / (h * .19f))
                                .toInt().coerceIn(0, 2)

                            val idx = row * 2 + col
                            val cp = zones[idx]

                            if (cp <= best || cp <= checkpoint) {
                                startAt(cp)
                            }
                        }
                    }

                    2 -> {
                        when {
                            x > w - 65 && y < 65 -> state = 3
                            x < 86 && y > h - 105 -> left = true
                            x < 165 && y > h - 105 -> right = true

                            x > w - 110 && y > h - 125 -> {
                                jumpHeld = true

                                if (vy == 0f ||
                                    levels[1] > 0 && jumpUsed < 1
                                ) {
                                    vy = -(8.4f + levels[0] * .8f)
                                    jumpUsed++
                                }
                            }

                            checkpoint > 0 &&
                            abs(heightM - checkpoint) < 8f &&
                            y < h * .18f -> state = 4
                        }
                    }

                    3 -> {
                        if (y in h * .38f..h * .50f) {
                            startAt(checkpoint)
                        } else if (y in h * .50f..h * .62f) {
                            state = 0
                        } else if (y in h * .62f..h * .76f) {
                            choosePet()
                        }
                    }

                    4 -> {
                        if (y > h * .77f) {
                            state = 2
                        } else {
                            val col = if (x > w * .5f) 1 else 0
                            val row = ((y - h * .39f) / (h * .065f))
                                .toInt()

                            val idx = row * 2 + col

                            if (idx in abilityNames.indices &&
                                coins >= 20 &&
                                levels[idx] < 5
                            ) {
                                coins -= 20
                                levels[idx]++
                                save()
                                state = 2
                            }
                        }
                    }

                    5 -> {
                        if (y > h * .80f) state = 0
                    }
                }

                invalidate()
                return true
            }

            MotionEvent.ACTION_UP,
            MotionEvent.ACTION_CANCEL -> {
                left = false
                right = false
                jumpHeld = false
                invalidate()
                return true
            }
        }

        return true
    }
}
