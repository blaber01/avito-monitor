package com.example.avitomonitor

import android.Manifest
import android.app.NotificationChannel
import android.app.NotificationManager
import android.content.Context
import android.content.pm.PackageManager
import android.os.Build
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Add
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import androidx.core.app.NotificationCompat
import androidx.core.app.NotificationManagerCompat
import androidx.core.content.ContextCompat

class MainActivity : ComponentActivity() {

    private val requestPermissionLauncher = registerForActivityResult(
        ActivityResultContracts.RequestPermission()
    ) { _ -> }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        createNotificationChannel()
        checkNotificationPermission()

        setContent {
            MaterialTheme {
                Surface(modifier = Modifier.fillMaxSize(), color = MaterialTheme.colorScheme.background) {
                    MainScreen(
                        onTestNotification = { title, desc ->
                            showNotification(this, title, desc)
                        }
                    )
                }
            }
        }
    }

    private fun checkNotificationPermission() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            if (ContextCompat.checkSelfPermission(this, Manifest.permission.POST_NOTIFICATIONS) !=
                PackageManager.PERMISSION_GRANTED
            ) {
                requestPermissionLauncher.launch(Manifest.permission.POST_NOTIFICATIONS)
            }
        }
    }

    private fun createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val name = "Avito Monitor Channel"
            val descriptionText = "Уведомления о новых объявлениях"
            val importance = NotificationManager.IMPORTANCE_HIGH
            val channel = NotificationChannel("avito_channel", name, importance).apply {
                description = descriptionText
            }
            val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
            notificationManager.createNotificationChannel(channel)
        }
    }
}

data class FilterFolder(
    val name: String,
    val queryUrl: String,
    val aiExclude: String,
    val count: Int
)

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun MainScreen(onTestNotification: (String, String) -> Unit) {
    var folders by remember {
        mutableStateOf(
            listOf(
                FilterFolder("Авто до 1.5 млн", "avito.ru/cars", "кроме битых", 3),
                FilterFolder("Видеокарты RTX", "avito.ru/gpus", "кроме майнинга", 12)
            )
        )
    }
    var showDialog by remember { mutableStateOf(false) }

    Scaffold(
        topBar = { TopAppBar(title = { Text("Монитор Авито") }) },
        floatingActionButton = {
            FloatingActionButton(onClick = { showDialog = true }) {
                Icon(Icons.Default.Add, contentDescription = "Добавить папки")
            }
        }
    ) { padding ->
        LazyColumn(modifier = Modifier.padding(padding).fillMaxSize()) {
            items(folders) { folder ->
                FolderCard(folder = folder) {
                    onTestNotification(folder.name, "Новое авто! Фильтр ИИ: ${folder.aiExclude}")
                }
            }
        }
    }

    if (showDialog) {
        AddFolderDialog(
            onDismiss = { showDialog = false },
            onSave = { name, url, exclude ->
                folders = folders + FilterFolder(name, url, exclude, 0)
                showDialog = false
            }
        )
    }
}

@Composable
fun FolderCard(folder: FilterFolder, onClick: () -> Unit) {
    Card(
        modifier = Modifier.fillMaxWidth().padding(8.dp),
        elevation = CardDefaults.cardElevation(4.dp),
        onClick = onClick
    ) {
        Column(modifier = Modifier.padding(16.dp)) {
            Text(text = folder.name, style = MaterialTheme.typography.titleMedium)
            Spacer(modifier = Modifier.height(4.dp))
            Text(text = "ИИ-исключение: ${folder.aiExclude.ifEmpty { "нет" }}", style = MaterialTheme.typography.bodySmall, color = Color.Gray)
            Spacer(modifier = Modifier.height(2.dp))
            Text(text = "Подходящих: ${folder.count}", style = MaterialTheme.typography.bodyMedium)
        }
    }
}

@Composable
fun AddFolderDialog(onDismiss: () -> Unit, onSave: (String, String, String) -> Unit) {
    var folderName by remember { mutableStateOf("") }
    var searchUrl by remember { mutableStateOf("") }
    var aiExclude by remember { mutableStateOf("") }

    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text("Новая папка поиска") },
        text = {
            Column {
                OutlinedTextField(value = folderName, onValueChange = { folderName = it }, label = { Text("Название папки") })
                Spacer(modifier = Modifier.height(8.dp))
                OutlinedTextField(value = searchUrl, onValueChange = { searchUrl = it }, label = { Text("Ссылка на поиск") })
                Spacer(modifier = Modifier.height(8.dp))
                OutlinedTextField(
                    value = aiExclude,
                    onValueChange = { aiExclude = it },
                    label = { Text("ИИ-фильтр (например: кроме битых)") }
                )
            }
        },
        confirmButton = {
            Button(onClick = { if (folderName.isNotBlank()) onSave(folderName, searchUrl, aiExclude) }) {
                Text("Добавить")
            }
        },
        dismissButton = {
            TextButton(onClick = onDismiss) { Text("Отмена") }
        }
    )
}

fun showNotification(context: Context, title: String, text: String) {
    val builder = NotificationCompat.Builder(context, "avito_channel")
        .setSmallIcon(android.R.drawable.ic_menu_search)
        .setContentTitle(title)
        .setContentText(text)
        .setPriority(NotificationCompat.PRIORITY_HIGH)
        .setAutoCancel(true)

    with(NotificationManagerCompat.from(context)) {
        if (androidx.core.app.ActivityCompat.checkSelfPermission(context, Manifest.permission.POST_NOTIFICATIONS) == PackageManager.PERMISSION_GRANTED) {
            notify(System.currentTimeMillis().toInt(), builder.build())
        }
    }
}

// Логика ИИ-фильтрации текстовых запросов («кроме...»)
fun passesAiFilter(itemTitle: String, userPrompt: String): Boolean {
    if (userPrompt.isBlank()) return true
    val excludeKeywords = userPrompt.lowercase()
        .replace("кроме", "")
        .replace("исключая", "")
        .trim()
        .split(" ", ",")
        .filter { it.isNotBlank() }

    val lowerTitle = itemTitle.lowercase()
    for (keyword in excludeKeywords) {
        if (lowerTitle.contains(keyword)) return false
    }
    return true
}
