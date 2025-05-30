package com.example.spincheck

import android.content.Context
import android.hardware.Sensor
import android.hardware.SensorEvent
import android.hardware.SensorEventListener
import android.hardware.SensorManager
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.compose.ui.Alignment
import androidx.compose.ui.draw.rotate
import com.example.spincheck.ui.theme.SpinCheckTheme
import android.view.WindowManager
import androidx.compose.ui.graphics.Color
import androidx.compose.foundation.background

class MainActivity : ComponentActivity(), SensorEventListener {

    private lateinit var sensorManager: SensorManager
    private var gyroscope: Sensor? = null

    private var onSensorData: ((Float, Float) -> Unit)? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        sensorManager = getSystemService(Context.SENSOR_SERVICE) as SensorManager
        gyroscope = sensorManager.getDefaultSensor(Sensor.TYPE_GYROSCOPE)

        setContent {


            window.addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)

            SpinCheckTheme {
                var rpm by remember { mutableStateOf(0f) }
                var angle by remember { mutableStateOf(0f) }

                // Sensor-Callback verknüpfen
                onSensorData = { newRpm, newAngle ->
                    val threshold = 0.1f
                    rpm = if (newRpm > threshold) {
                        // Auf eine Nachkommastelle runden, letzte Zahl immer 0
                        (newRpm * 10).toInt() / 10f
                    } else {
                        0f
                    }
                    angle += newAngle
                }

                Box(
                    modifier = Modifier
                        .fillMaxSize()
                        .background(Color.Black)
                        .padding(32.dp),
                    contentAlignment = Alignment.Center
                ) {
                    Text(
                        text = "U/minute: \n %.1f".format(rpm),
                        modifier = Modifier.rotate(angle),
                        color = Color.White,
                        style = MaterialTheme.typography.displayLarge
                    )
                }
            }
        }
    }

    override fun onResume() {
        super.onResume()
        gyroscope?.let {
            sensorManager.registerListener(this, it, SensorManager.SENSOR_DELAY_GAME)
        }
    }

    override fun onPause() {
        super.onPause()
        sensorManager.unregisterListener(this)
    }

    override fun onSensorChanged(event: SensorEvent?) {
        if (event?.sensor?.type == Sensor.TYPE_GYROSCOPE) {
            val rotationZ = event.values[2] // rad/s um Z-Achse

            val rpm = (rotationZ * 60) / (2 * Math.PI)

            // Integration zur Rotation (sehr vereinfacht, 0.02 = Zeitfaktor)
            val angleDelta = (Math.toDegrees(rotationZ.toDouble()) * 0.02).toFloat()

            // RPM immer positiv anzeigen, aber Drehrichtung behalten für Animation
            onSensorData?.invoke(kotlin.math.abs(rpm.toFloat()), angleDelta)
        }
    }

    override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {}
}
