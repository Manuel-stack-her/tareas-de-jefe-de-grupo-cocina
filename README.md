<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gestor de Tareas</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    
    <!-- Librería para leer Excel -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
</head>
<body class="bg-gray-100 font-sans">

    <div class="max-w-md mx-auto bg-white min-h-screen shadow-lg">
        <!-- Header -->
        <div class="bg-gradient-to-r from-blue-500 to-blue-700 text-white p-6">
            <h1 class="text-2xl font-bold flex items-center">
                <i class="fas fa-tasks mr-3"></i> Mis Tareas
            </h1>
            <p class="text-sm opacity-90">Organiza y registra tu trabajo</p>
        </div>

        <!-- SECCIÓN NUEVA: CARGAR EXCEL -->
        <div class="p-6 border-b bg-green-50">
            <h2 class="text-lg font-semibold mb-3 text-green-700">
                <i class="fas fa-file-excel"></i> Cargar desde Excel
            </h2>
            <p class="text-xs text-gray-500 mb-3">
                Pon las tareas en la Columna A de tu Excel.
            </p>
            <input type="file" id="fileInput" accept=".xlsx, .xls" 
                class="w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-lg file:border-0 file:bg-green-100 file:text-green-700 hover:file:bg-green-200 mb-2">
            <button onclick="importarExcel()" 
                class="w-full bg-green-600 text-white py-2 rounded-lg font-semibold hover:bg-green-700 transition duration-300">
                📥 Importar Tareas
            </button>
        </div>

        <!-- Formulario Agregar Tarea -->
        <div class="p-6 border-b">
            <h2 class="text-lg font-semibold mb-4 text-gray-700">➕ Agregar Manualmente</h2>
            
            <input type="text" id="taskInput" placeholder="📝 Escribe el nombre de la tarea..." 
                class="w-full p-3 border border-gray-300 rounded-lg mb-3 focus:outline-none focus:ring-2 focus:ring-blue-500">

            <input type="file" id="imageInput" accept="image/*" 
                class="w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-lg file:border-0 file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100 mb-3">

            <button onclick="addTask()" 
                class="w-full bg-blue-600 text-white py-3 rounded-lg font-semibold hover:bg-blue-700 transition duration-300">
                Guardar Tarea
            </button>
        </div>

        <!-- Lista de Tareas -->
        <div class="p-6">
            <h2 class="text-lg font-semibold mb-4 text-gray-700">📋 Lista de Tareas</h2>
            <ul id="taskList" class="space-y-4"></ul>
        </div>
    </div>

    <script>
        let tasks = JSON.parse(localStorage.getItem('tasks')) || [];
        renderTasks();

        // 📥 FUNCIÓN PARA LEER EXCEL
        function importarExcel() {
            const input = document.getElementById('fileInput');
            if (!input.files || !input.files[0]) {
                alert('⚠️ Por favor selecciona un archivo Excel primero');
                return;
            }

            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const data = new Uint8Array(e.target.result);
                    const workbook = XLSX.read(data, {type: 'array'});
                    const firstSheetName = workbook.SheetNames[0];
                    const worksheet = workbook.Sheets[firstSheetName];
                    
                    // Convertir a JSON
                    const jsonData = XLSX.utils.sheet_to_json(worksheet, {header:1});
                    
                    let contador = 0;
                    // Recorrer filas (tomamos solo la columna A)
                    jsonData.forEach(row => {
                        if(row.length > 0 && row[0]) { // Si hay dato en columna A
                            const nuevaTarea = {
                                id: Date.now() + Math.random(),
                                text: row[0].toString(),
                                time: new Date().toLocaleString('es-ES'),
                                image: null
                            };
                            tasks.unshift(nuevaTarea);
                            contador++;
                        }
                    });

                    localStorage.setItem('tasks', JSON.stringify(tasks));
                    renderTasks();
                    input.value = ''; // Limpiar
                    alert(`✅ Éxito! Se cargaron ${contador} tareas desde tu Excel`);

                } catch (error) {
                    alert('❌ Error al leer el archivo. Asegúrate que sea un Excel válido.');
                    console.error(error);
                }
            };
            reader.readAsArrayBuffer(input.files[0]);
        }

        function addTask() {
            const taskText = document.getElementById('taskInput').value;
            const imageInput = document.getElementById('imageInput');
            
            if (!taskText) {
                alert('Por favor escribe una tarea');
                return;
            }

            const now = new Date();
            const dateTime = now.toLocaleString('es-ES', {
                day: '2-digit', month: '2-digit', year: 'numeric',
                hour: '2-digit', minute: '2-digit'
            });

            const newTask = {
                id: Date.now(),
                text: taskText,
                time: dateTime,
                image: null
            };

            if (imageInput.files && imageInput.files[0]) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    newTask.image = e.target.result;
                    saveAndRender(newTask);
                };
                reader.readAsDataURL(imageInput.files[0]);
            } else {
                saveAndRender(newTask);
            }

            document.getElementById('taskInput').value = '';
            imageInput.value = '';
        }

        function saveAndRender(task) {
            tasks.unshift(task);
            localStorage.setItem('tasks', JSON.stringify(tasks));
            renderTasks();
        }

        function deleteTask(id) {
            tasks = tasks.filter(task => task.id !== id);
            localStorage.setItem('tasks', JSON.stringify(tasks));
            renderTasks();
        }

        function renderTasks() {
            const list = document.getElementById('taskList');
            if (tasks.length === 0) {
                list.innerHTML = '<p class="text-center text-gray-400 py-10">No hay tareas aún. ¡Agrega una o carga tu Excel!</p>';
                return;
            }

            list.innerHTML = tasks.map(task => `
                <li class="bg-gray-50 p-4 rounded-xl border border-gray-200 shadow-sm">
                    <div class="flex justify-between items-start">
                        <div class="flex-1">
                            <p class="font-medium text-gray-800 text-base">${task.text}</p>
                            <p class="text-xs text-gray-500 mt-1 flex items-center">
                                <i class="fas fa-clock mr-1"></i> ${task.time}
                            </p>
                        </div>
                        <button onclick="deleteTask(${task.id})" class="text-red-400 hover:text-red-600 ml-2">
                            <i class="fas fa-trash"></i>
                        </button>
                    </div>
                    
                    ${task.image ? `
                        <div class="mt-3">
                            <img src="${task.image}" alt="Evidencia" class="rounded-lg w-full h-40 object-cover border">
                            <p class="text-xs text-gray-500 mt-1"><i class="fas fa-image mr-1"></i> Foto adjuntada</p>
                        </div>
                    ` : ''}
                </li>
            `).join('');
        }
    </script>

</body>
</html>

