<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>GPS Tracker App</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://api.mapbox.com/mapbox-gl-js/v2.14.1/mapbox-gl.css" rel="stylesheet">
    <script src="https://api.mapbox.com/mapbox-gl-js/v2.14.1/mapbox-gl.js"></script>
    <style id="app-style">
        body {
            font-family: 'Inter', sans-serif;
            height: 100vh;
            margin: 0;
            padding: 0;
            overflow: hidden;
            position: relative;
        }

        #app {
            display: flex;
            flex-direction: column;
            height: 100vh;
        }

        #content {
            flex-grow: 1;
            overflow-y: auto;
            padding-bottom: 70px; /* Space for nav bar */
        }

        #map {
            width: 100%;
            height: 100%;
        }

        .nav-item {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            flex: 1;
            text-decoration: none;
            color: #718096;
            transition: all 0.3s;
            padding: 10px 0;
        }

        .nav-item.active {
            color: #4C51BF;
        }

        .nav-item i {
            font-size: 1.5rem;
            margin-bottom: 4px;
        }

        .nav-item span {
            font-size: 0.75rem;
        }

        .screen {
            height: 100%;
            width: 100%;
            display: none;
            overflow-y: auto;
        }

        .screen.active {
            display: block;
        }

        .device-card {
            transition: transform 0.2s;
        }

        .device-card:hover {
            transform: translateY(-5px);
        }

        .modal {
            display: none;
            position: fixed;
            z-index: 1000;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
        }

        .modal-content {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 80%;
            max-width: 500px;
            background: white;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
        }

        .close-modal {
            position: absolute;
            right: 15px;
            top: 10px;
            font-size: 1.5rem;
            cursor: pointer;
        }

        /* Marker */
        .mapboxgl-marker {
            width: 25px;
            height: 25px;
            background-color: #4338ca;
            border: 2px solid white;
            border-radius: 50%;
            box-shadow: 0 0 10px rgba(0,0,0,0.3);
        }

        /* Pulse Effect */
        .pulse {
            display: block;
            width: 25px;
            height: 25px;
            border-radius: 50%;
            background: #4338ca;
            cursor: pointer;
            box-shadow: 0 0 0 rgba(67, 56, 202, 0.4);
            animation: pulse 2s infinite;
        }

        @keyframes pulse {
            0% {
                box-shadow: 0 0 0 0 rgba(67, 56, 202, 0.4);
            }
            70% {
                box-shadow: 0 0 0 15px rgba(67, 56, 202, 0);
            }
            100% {
                box-shadow: 0 0 0 0 rgba(67, 56, 202, 0);
            }
        }

        /* Loading spinner */
        .loading-spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            width: 36px;
            height: 36px;
            border-radius: 50%;
            border-left-color: #4C51BF;
            animation: spin 1s linear infinite;
        }

        @keyframes spin {
            0% {
                transform: rotate(0deg);
            }
            100% {
                transform: rotate(360deg);
            }
        }
    </style>
</head>
<body class="bg-gray-100">
    <div id="app">
        <div id="content">
            <!-- MAPA SCREEN -->
            <div id="mapa-screen" class="screen active">
                <div id="map" class="w-full h-full"></div>
                <div id="location-permission" class="hidden fixed top-0 left-0 w-full h-full bg-black bg-opacity-80 flex flex-col items-center justify-center text-white p-5 text-center z-50">
                    <i class="fas fa-map-marker-alt text-5xl mb-4 text-indigo-500"></i>
                    <h2 class="text-2xl font-bold mb-2">Permiso de ubicación</h2>
                    <p class="mb-6">Esta aplicación necesita acceder a tu ubicación para mostrar el mapa en tiempo real</p>
                    <button id="enable-location" class="bg-indigo-600 text-white py-3 px-6 rounded-lg text-lg font-medium hover:bg-indigo-700 transition duration-200">
                        Permitir acceso
                    </button>
                </div>
            </div>

            <!-- REPORTE SCREEN -->
            <div id="reporte-screen" class="screen p-5">
                <h2 class="text-2xl font-bold mb-6 text-gray-800">Reporte</h2>
                
                <div class="grid grid-cols-1 gap-6">
                    <button id="btn-report-theft" class="bg-red-600 hover:bg-red-700 text-white p-6 rounded-lg shadow-md flex items-center justify-center transition">
                        <i class="fas fa-exclamation-triangle mr-3 text-2xl"></i>
                        <span class="text-xl font-medium">Reportar celular robado</span>
                    </button>
                    
                    <button id="btn-contact-agents" class="bg-blue-600 hover:bg-blue-700 text-white p-6 rounded-lg shadow-md flex items-center justify-center transition">
                        <i class="fas fa-headset mr-3 text-2xl"></i>
                        <span class="text-xl font-medium">Contacto con agentes</span>
                    </button>
                </div>

                <!-- Report Theft Modal -->
                <div id="report-theft-modal" class="modal">
                    <div class="modal-content">
                        <span class="close-modal" data-modal="report-theft-modal">&times;</span>
                        <h3 class="text-xl font-bold mb-4 text-gray-800">Reportar celular robado</h3>
                        <form id="theft-report-form">
                            <div class="mb-4">
                                <label for="imei" class="block text-sm font-medium text-gray-700 mb-1">IMEI del dispositivo</label>
                                <input type="text" id="imei" name="imei" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500" placeholder="15 dígitos" required pattern="\d{15}">
                                <p class="text-xs text-gray-500 mt-1">*Marca *#06# en el teclado de marcación para ver tu IMEI</p>
                            </div>
                            <div class="mb-4">
                                <label for="description" class="block text-sm font-medium text-gray-700 mb-1">Descripción</label>
                                <textarea id="description" name="description" rows="3" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500" placeholder="Detalles del incidente" required></textarea>
                            </div>
                            <div class="mb-4">
                                <label for="date" class="block text-sm font-medium text-gray-700 mb-1">Fecha del incidente</label>
                                <input type="date" id="date" name="date" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                            </div>
                            <div class="mb-4">
                                <label for="location" class="block text-sm font-medium text-gray-700 mb-1">Ubicación</label>
                                <input type="text" id="location" name="location" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500" placeholder="Dirección o referencia" required>
                            </div>
                            <button type="submit" class="w-full bg-red-600 text-white py-2 px-4 rounded-md hover:bg-red-700 transition focus:outline-none focus:ring-2 focus:ring-red-500">
                                Enviar reporte
                            </button>
                        </form>
                    </div>
                </div>

                <!-- Contact Agents Modal -->
                <div id="contact-agents-modal" class="modal">
                    <div class="modal-content">
                        <span class="close-modal" data-modal="contact-agents-modal">&times;</span>
                        <h3 class="text-xl font-bold mb-4 text-gray-800">Contacto con agentes</h3>
                        <div class="grid grid-cols-1 gap-4">
                            <a href="tel:+51987654321" class="bg-green-600 text-white p-4 rounded-lg flex items-center justify-center hover:bg-green-700 transition">
                                <i class="fas fa-phone-alt mr-2"></i>
                                <span>Llamar a un agente</span>
                            </a>
                            <button id="start-chat" class="bg-blue-600 text-white p-4 rounded-lg flex items-center justify-center hover:bg-blue-700 transition">
                                <i class="fas fa-comment-dots mr-2"></i>
                                <span>Iniciar chat</span>
                            </button>
                        </div>

                        <div id="chat-container" class="hidden mt-4 border border-gray-300 rounded-lg overflow-hidden">
                            <div class="bg-gray-100 p-3 border-b border-gray-300">
                                <div class="flex items-center">
                                    <div class="w-2 h-2 bg-green-500 rounded-full mr-2"></div>
                                    <span class="font-medium">Agente disponible</span>
                                </div>
                            </div>
                            <div id="chat-messages" class="p-3 h-60 overflow-y-auto bg-white">
                                <div class="flex mb-3">
                                    <div class="bg-blue-100 p-2 rounded-lg max-w-[80%]">
                                        <p class="text-sm">Hola, soy el agente virtual. ¿En qué puedo ayudarte hoy?</p>
                                        <span class="text-xs text-gray-500">10:30</span>
                                    </div>
                                </div>
                            </div>
                            <div class="p-3 border-t border-gray-300 bg-white">
                                <div class="flex">
                                    <input type="text" id="chat-input" placeholder="Escribe tu mensaje..." class="flex-grow px-3 py-2 border border-gray-300 rounded-l-md focus:outline-none focus:ring-1 focus:ring-indigo-500">
                                    <button id="send-message" class="bg-indigo-600 text-white px-4 py-2 rounded-r-md hover:bg-indigo-700">
                                        <i class="fas fa-paper-plane"></i>
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- PROMOCIONES SCREEN -->
            <div id="promociones-screen" class="screen p-5">
                <h2 class="text-2xl font-bold mb-6 text-gray-800">Promociones</h2>
                
                <div class="grid grid-cols-1 sm:grid-cols-2 gap-5" id="devices-container">
                    <div class="bg-white rounded-lg shadow-md overflow-hidden device-card">
                        <img src="https://cdn.pixabay.com/photo/2016/12/09/11/33/smartphone-1894723_1280.jpg" alt="Smartphone" class="w-full h-48 object-cover">
                        <div class="p-4">
                            <h3 class="text-lg font-semibold mb-2">Samsung Galaxy S25</h3>
                            <div class="flex items-center mb-2">
                                <div class="flex text-yellow-400">
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star-half-alt"></i>
                                </div>
                                <span class="text-xs text-gray-500 ml-2">(128)</span>
                            </div>
                            <p class="text-gray-600 text-sm mb-3">
                                Pantalla 6.8", 16GB RAM, 512GB almacenamiento, cámara 108MP
                            </p>
                            <div class="flex justify-between items-center">
                                <span class="text-xl font-bold text-indigo-700">S/ 2,499</span>
                                <span class="text-sm line-through text-gray-500">S/ 3,199</span>
                            </div>
                        </div>
                    </div>

                    <div class="bg-white rounded-lg shadow-md overflow-hidden device-card">
                        <img src="https://cdn.pixabay.com/photo/2014/08/05/10/30/iphone-410324_1280.jpg" alt="iPhone" class="w-full h-48 object-cover">
                        <div class="p-4">
                            <h3 class="text-lg font-semibold mb-2">iPhone 17 Pro</h3>
                            <div class="flex items-center mb-2">
                                <div class="flex text-yellow-400">
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                </div>
                                <span class="text-xs text-gray-500 ml-2">(207)</span>
                            </div>
                            <p class="text-gray-600 text-sm mb-3">
                                Pantalla 6.5", 12GB RAM, 256GB almacenamiento, cámara 48MP
                            </p>
                            <div class="flex justify-between items-center">
                                <span class="text-xl font-bold text-indigo-700">S/ 3,999</span>
                                <span class="text-sm line-through text-gray-500">S/ 4,599</span>
                            </div>
                        </div>
                    </div>

                    <div class="bg-white rounded-lg shadow-md overflow-hidden device-card">
                        <img src="https://cdn.pixabay.com/photo/2017/08/11/14/19/honor-2631271_1280.jpg" alt="Xiaomi" class="w-full h-48 object-cover">
                        <div class="p-4">
                            <h3 class="text-lg font-semibold mb-2">Xiaomi Mi 14 Lite</h3>
                            <div class="flex items-center mb-2">
                                <div class="flex text-yellow-400">
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                    <i class="far fa-star"></i>
                                </div>
                                <span class="text-xs text-gray-500 ml-2">(84)</span>
                            </div>
                            <p class="text-gray-600 text-sm mb-3">
                                Pantalla 6.7", 8GB RAM, 128GB almacenamiento, cámara 64MP
                            </p>
                            <div class="flex justify-between items-center">
                                <span class="text-xl font-bold text-indigo-700">S/ 1,299</span>
                                <span class="text-sm line-through text-gray-500">S/ 1,799</span>
                            </div>
                        </div>
                    </div>

                    <div class="bg-white rounded-lg shadow-md overflow-hidden device-card">
                        <img src="https://cdn.pixabay.com/photo/2018/01/08/02/34/technology-3068617_1280.jpg" alt="Google Pixel" class="w-full h-48 object-cover">
                        <div class="p-4">
                            <h3 class="text-lg font-semibold mb-2">Google Pixel 9</h3>
                            <div class="flex items-center mb-2">
                                <div class="flex text-yellow-400">
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star"></i>
                                    <i class="fas fa-star-half-alt"></i>
                                </div>
                                <span class="text-xs text-gray-500 ml-2">(93)</span>
                            </div>
                            <p class="text-gray-600 text-sm mb-3">
                                Pantalla 6.4", 12GB RAM, 256GB almacenamiento, cámara 50MP
                            </p>
                            <div class="flex justify-between items-center">
                                <span class="text-xl font-bold text-indigo-700">S/ 2,899</span>
                                <span class="text-sm line-through text-gray-500">S/ 3,499</span>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- PERFIL SCREEN -->
            <div id="perfil-screen" class="screen p-5">
                <h2 class="text-2xl font-bold mb-6 text-gray-800">Perfil</h2>
                
                <div id="profile-guest" class="grid grid-cols-1 gap-6">
                    <div id="btn-login" class="bg-indigo-600 hover:bg-indigo-700 text-white p-5 rounded-lg shadow-md cursor-pointer transition flex items-center">
                        <div class="bg-white bg-opacity-20 w-12 h-12 rounded-full flex items-center justify-center mr-4">
                            <i class="fas fa-sign-in-alt text-xl"></i>
                        </div>
                        <div>
                            <h3 class="font-semibold text-lg">Iniciar sesión</h3>
                            <p class="text-sm text-indigo-100">Accede a tu cuenta</p>
                        </div>
                        <i class="fas fa-chevron-right ml-auto"></i>
                    </div>
                    
                    <div id="btn-register" class="bg-blue-600 hover:bg-blue-700 text-white p-5 rounded-lg shadow-md cursor-pointer transition flex items-center">
                        <div class="bg-white bg-opacity-20 w-12 h-12 rounded-full flex items-center justify-center mr-4">
                            <i class="fas fa-user-plus text-xl"></i>
                        </div>
                        <div>
                            <h3 class="font-semibold text-lg">Registrarse</h3>
                            <p class="text-sm text-blue-100">Crea una cuenta nueva</p>
                        </div>
                        <i class="fas fa-chevron-right ml-auto"></i>
                    </div>
                    
                    <div id="btn-agent-mode" class="bg-yellow-600 hover:bg-yellow-700 text-white p-5 rounded-lg shadow-md cursor-pointer transition flex items-center">
                        <div class="bg-white bg-opacity-20 w-12 h-12 rounded-full flex items-center justify-center mr-4">
                            <i class="fas fa-user-shield text-xl"></i>
                        </div>
                        <div>
                            <h3 class="font-semibold text-lg">Modo Agente</h3>
                            <p class="text-sm text-yellow-100">Acceso para agentes autorizados</p>
                        </div>
                        <i class="fas fa-chevron-right ml-auto"></i>
                    </div>
                    
                    <div id="btn-premium" class="bg-purple-600 hover:bg-purple-700 text-white p-5 rounded-lg shadow-md cursor-pointer transition flex items-center">
                        <div class="bg-white bg-opacity-20 w-12 h-12 rounded-full flex items-center justify-center mr-4">
                            <i class="fas fa-crown text-xl"></i>
                        </div>
                        <div>
                            <h3 class="font-semibold text-lg">Premium</h3>
                            <p class="text-sm text-purple-100">Accede a funciones exclusivas</p>
                        </div>
                        <i class="fas fa-chevron-right ml-auto"></i>
                    </div>
                </div>

                <!-- Login Modal -->
                <div id="login-modal" class="modal">
                    <div class="modal-content">
                        <span class="close-modal" data-modal="login-modal">&times;</span>
                        <h3 class="text-xl font-bold mb-4 text-gray-800">Iniciar sesión</h3>
                        <form id="login-form">
                            <div class="mb-4">
                                <label for="login-email" class="block text-sm font-medium text-gray-700 mb-1">Correo electrónico</label>
                                <input type="email" id="login-email" name="email" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                            </div>
                            <div class="mb-4">
                                <label for="login-password" class="block text-sm font-medium text-gray-700 mb-1">Contraseña</label>
                                <input type="password" id="login-password" name="password" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                            </div>
                            <div class="mb-4 flex items-center">
                                <input type="checkbox" id="remember-me" name="remember-me" class="h-4 w-4 text-indigo-600 focus:ring-indigo-500 border-gray-300 rounded">
                                <label for="remember-me" class="ml-2 block text-sm text-gray-700">
                                    Recordarme
                                </label>
                                <a href="javascript:void(0)" class="text-sm text-indigo-600 hover:text-indigo-500 ml-auto">
                                    ¿Olvidaste tu contraseña?
                                </a>
                            </div>
                            <button type="submit" class="w-full bg-indigo-600 text-white py-2 px-4 rounded-md hover:bg-indigo-700 transition focus:outline-none focus:ring-2 focus:ring-indigo-500">
                                Iniciar sesión
                            </button>
                        </form>
                    </div>
                </div>

                <!-- Register Modal -->
                <div id="register-modal" class="modal">
                    <div class="modal-content">
                        <span class="close-modal" data-modal="register-modal">&times;</span>
                        <h3 class="text-xl font-bold mb-4 text-gray-800">Registrarse</h3>
                        <form id="register-form">
                            <div class="mb-4">
                                <label for="register-name" class="block text-sm font-medium text-gray-700 mb-1">Nombre completo</label>
                                <input type="text" id="register-name" name="name" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                            </div>
                            <div class="mb-4">
                                <label for="register-email" class="block text-sm font-medium text-gray-700 mb-1">Correo electrónico</label>
                                <input type="email" id="register-email" name="email" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required>
                            </div>
                            <div class="mb-4">
                                <label for="register-password" class="block text-sm font-medium text-gray-700 mb-1">Contraseña</label>
                                <input type="password" id="register-password" name="password" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required minlength="8">
                                <p class="text-xs text-gray-500 mt-1">Mínimo 8 caracteres</p>
                            </div>
                            <div class="mb-4">
                                <label for="register-confirm-password" class="block text-sm font-medium text-gray-700 mb-1">Confirmar contraseña</label>
                                <input type="password" id="register-confirm-password" name="confirm-password" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500" required minlength="8">
                            </div>
                            <div class="mb-4 flex items-center">
                                <input type="checkbox" id="terms" name="terms" class="h-4 w-4 text-indigo-600 focus:ring-indigo-500 border-gray-300 rounded" required>
                                <label for="terms" class="ml-2 block text-sm text-gray-700">
                                    Acepto los <a href="javascript:void(0)" class="text-indigo-600 hover:text-indigo-500">términos y condiciones</a>
                                </label>
                            </div>
                            <button type="submit" class="w-full bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700 transition focus:outline-none focus:ring-2 focus:ring-blue-500">
                                Crear cuenta
                            </button>
                        </form>
                    </div>
                </div>

                <!-- Agent Mode Modal -->
                <div id="agent-modal" class="modal">
                    <div class="modal-content">
                        <span class="close-modal" data-modal="agent-modal">&times;</span>
                        <h3 class="text-xl font-bold mb-4 text-gray-800">Modo Agente</h3>
                        <div class="flex flex-col items-center justify-center">
                            <div class="w-20 h-20 rounded-full bg-yellow-100 flex items-center justify-center mb-4">
                                <i class="fas fa-user-shield text-yellow-600 text-3xl"></i>
                            </div>
                            <p class="text-center text-gray-600 mb-4">
                                El modo agente está disponible solo para personal autorizado. Ingrese sus credenciales para continuar.
                            </p>
                            <form id="agent-form" class="w-full">
                                <div class="mb-4">
                                    <label for="agent-id" class="block text-sm font-medium text-gray-700 mb-1">ID de Agente</label>
                                    <input type="text" id="agent-id" name="agent-id" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-yellow-500" required>
                                </div>
                                <div class="mb-4">
                                    <label for="agent-password" class="block text-sm font-medium text-gray-700 mb-1">Contraseña</label>
                                    <input type="password" id="agent-password" name="agent-password" class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-yellow-500" required>
                                </div>
                                <button type="submit" class="w-full bg-yellow-600 text-white py-2 px-4 rounded-md hover:bg-yellow-700 transition focus:outline-none focus:ring-2 focus:ring-yellow-500">
                                    Acceder
                                </button>
                            </form>
                        </div>
                    </div>
                </div>

                <!-- Premium Modal -->
                <div id="premium-modal" class="modal">
                    <div class="modal-content">
                        <span class="close-modal" data-modal="premium-modal">&times;</span>
                        <div class="text-center mb-4">
                            <h3 class="text-xl font-bold mb-1 text-gray-800">Suscripción Premium</h3>
                            <p class="text-gray-600">Desbloquea todas las funciones</p>
                        </div>

                        <div class="bg-gradient-to-br from-purple-600 to-indigo-800 rounded-lg p-5 text-white mb-6">
                            <div class="flex items-center mb-3">
                                <i class="fas fa-crown text-yellow-400 text-2xl mr-3"></i>
                                <h4 class="text-lg font-semibold">Plan Premium</h4>
                            </div>
                            <ul class="space-y-2 mb-4">
                                <li class="flex items-start">
                                    <i class="fas fa-check-circle text-green-300 mt-1 mr-2"></i>
                                    <span>Seguimiento de ubicación en tiempo real</span>
                                </li>
                                <li class="flex items-start">
                                    <i class="fas fa-check-circle text-green-300 mt-1 mr-2"></i>
                                    <span>Alertas de seguridad instantáneas</span>
                                </li>
                                <li class="flex items-start">
                                    <i class="fas fa-check-circle text-green-300 mt-1 mr-2"></i>
                                    <span>Soporte prioritario 24/7</span>
                                </li>
                                <li class="flex items-start">
                                    <i class="fas fa-check-circle text-green-300 mt-1 mr-2"></i>
                                    <span>Reportes avanzados y análisis de datos</span>
                                </li>
                            </ul>
                            <div class="flex items-center justify-between">
                                <div>
                                    <span class="text-2xl font-bold">S/ 19.99</span>
                                    <span class="text-sm">/mes</span>
                                </div>
                                <button class="bg-white text-purple-700 font-semibold py-2 px-4 rounded hover:bg-gray-100 transition">
                                    Suscribirse
                                </button>
                            </div>
                        </div>

                        <div class="bg-gray-100 rounded-lg p-4">
                            <h5 class="font-semibold mb-2">También disponible</h5>
                            <div class="flex items-center justify-between">
                                <div>
                                    <h6 class="font-medium">Plan Anual</h6>
                                    <p class="text-sm text-gray-600">S/ 199.99 (2 meses gratis)</p>
                                </div>
                                <button class="bg-indigo-100 text-indigo-700 font-medium py-1 px-3 rounded hover:bg-indigo-200 transition text-sm">
                                    Ver detalles
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- NAVIGATION BAR -->
        <div class="fixed bottom-0 left-0 right-0 bg-white border-t border-gray-200 flex justify-around items-center h-16 z-10">
            <a href="javascript:void(0)" class="nav-item active" data-screen="mapa-screen">
                <i class="fas fa-map-marker-alt"></i>
                <span>Mapa</span>
            </a>
            <a href="javascript:void(0)" class="nav-item" data-screen="reporte-screen">
                <i class="fas fa-exclamation-circle"></i>
                <span>Reporte</span>
            </a>
            <a href="javascript:void(0)" class="nav-item" data-screen="promociones-screen">
                <i class="fas fa-tag"></i>
                <span>Promociones</span>
            </a>
            <a href="javascript:void(0)" class="nav-item" data-screen="perfil-screen">
                <i class="fas fa-user"></i>
                <span>Perfil</span>
            </a>
        </div>
    </div>

    <script id="app-script">
        document.addEventListener('DOMContentLoaded', function() {
            // Initialize navigation
            initNavigation();
            
            // Initialize modals
            initModals();
            
            // Initialize map
            initMap();

            // Initialize chat
            initChat();

            // Initialize forms
            initForms();
        });

        // Navigation between screens
        function initNavigation() {
            const navItems = document.querySelectorAll('.nav-item');
            const screens = document.querySelectorAll('.screen');
            
            navItems.forEach(item => {
                item.addEventListener('click', function() {
                    const targetScreen = this.getAttribute('data-screen');
                    
                    // Update active nav item
                    navItems.forEach(navItem => navItem.classList.remove('active'));
                    this.classList.add('active');
                    
                    // Show selected screen
                    screens.forEach(screen => {
                        screen.classList.remove('active');
                        if (screen.id === targetScreen) {
                            screen.classList.add('active');
                        }
                    });
                });
            });
        }

        // Modal functionality
        function initModals() {
            // Profile screen buttons
            document.getElementById('btn-login').addEventListener('click', () => openModal('login-modal'));
            document.getElementById('btn-register').addEventListener('click', () => openModal('register-modal'));
            document.getElementById('btn-agent-mode').addEventListener('click', () => openModal('agent-modal'));
            document.getElementById('btn-premium').addEventListener('click', () => openModal('premium-modal'));
            
            // Report screen buttons
            document.getElementById('btn-report-theft').addEventListener('click', () => openModal('report-theft-modal'));
            document.getElementById('btn-contact-agents').addEventListener('click', () => openModal('contact-agents-modal'));
            
            // Close buttons
            document.querySelectorAll('.close-modal').forEach(closeBtn => {
                closeBtn.addEventListener('click', function() {
                    const modalId = this.getAttribute('data-modal');
                    closeModal(modalId);
                });
            });

            // Close modal when clicking outside
            document.querySelectorAll('.modal').forEach(modal => {
                modal.addEventListener('click', function(e) {
                    if (e.target === this) {
                        closeModal(this.id);
                    }
                });
            });
        }

        function openModal(modalId) {
            document.getElementById(modalId).style.display = 'block';
            document.body.style.overflow = 'hidden';
        }

        function closeModal(modalId) {
            document.getElementById(modalId).style.display = 'none';
            document.body.style.overflow = 'auto';
        }

        // Map functionality
        function initMap() {
            // Check if geolocation permission is needed
            if (!navigator.geolocation) {
                document.getElementById('location-permission').classList.remove('hidden');
                return;
            }

            // Show permission dialog
            document.getElementById('location-permission').classList.remove('hidden');

            // Handle permission button click
            document.getElementById('enable-location').addEventListener('click', function() {
                document.getElementById('location-permission').classList.add('hidden');
                
                // Initialize map with Mapbox
                mapboxgl.accessToken = 'pk.eyJ1IjoibWFwYm94IiwiYSI6ImNpejY4M29iazA2Z2gycXA4N2pmbDZmangifQ.-g_vE53SD2WrJ6tFX7QHmA';
                const map = new mapboxgl.Map({
                    container: 'map',
                    style: 'mapbox://styles/mapbox/streets-v11',
                    center: [-77.0428, -12.0464], // Lima, Peru as default
                    zoom: 15
                });

                // Add controls
                map.addControl(new mapboxgl.NavigationControl());

                // Add marker for user position
                const marker = new mapboxgl.Marker({
                    element: createMarkerElement()
                });

                // Watch position and update map
                if (navigator.geolocation) {
                    navigator.geolocation.watchPosition(
                        function(position) {
                            const { latitude, longitude } = position.coords;
                            marker.setLngLat([longitude, latitude]).addTo(map);
                            map.flyTo({
                                center: [longitude, latitude],
                                essential: true
                            });
                        },
                        function(error) {
                            console.error('Error getting location:', error);
                        }
                    );
                }
            });
        }

        function createMarkerElement() {
            const el = document.createElement('div');
            el.className = 'pulse';
            return el;
        }

        // Chat functionality
        function initChat() {
            const startChatButton = document.getElementById('start-chat');
            const chatContainer = document.getElementById('chat-container');
            const chatInput = document.getElementById('chat-input');
            const sendButton = document.getElementById('send-message');
            const chatMessages = document.getElementById('chat-messages');

            startChatButton.addEventListener('click', function() {
                chatContainer.classList.remove('hidden');
                this.classList.add('hidden');
            });

            sendButton.addEventListener('click', sendMessage);
            chatInput.addEventListener('keypress', function(e) {
                if (e.key === 'Enter') {
                    sendMessage();
                }
            });

            function sendMessage() {
                const message = chatInput.value.trim();
                if (message) {
                    // Add user message
                    const time = new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
                    const userMessageHTML = `
                        <div class="flex mb-3 justify-end">
                            <div class="bg-indigo-100 p-2 rounded-lg max-w-[80%]">
                                <p class="text-sm">${message}</p>
                                <span class="text-xs text-gray-500">${time}</span>
                            </div>
                        </div>
                    `;
                    chatMessages.insertAdjacentHTML('beforeend', userMessageHTML);
                    
                    // Clear input
                    chatInput.value = '';
                    
                    // Auto scroll to bottom
                    chatMessages.scrollTop = chatMessages.scrollHeight;
                    
                    // Simulate agent response after delay
                    setTimeout(() => {
                        const agentMessageHTML = `
                            <div class="flex mb-3">
                                <div class="bg-blue-100 p-2 rounded-lg max-w-[80%]">
                                    <p class="text-sm">Este es un prototipo. La funcionalidad completa de chat será implementada en la versión final.</p>
                                    <span class="text-xs text-gray-500">${new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})}</span>
                                </div>
                            </div>
                        `;
                        chatMessages.insertAdjacentHTML('beforeend', agentMessageHTML);
                        chatMessages.scrollTop = chatMessages.scrollHeight;
                    }, 1000);
                }
            }
        }

        // Form submission handling
        function initForms() {
            // Login form
            document.getElementById('login-form').addEventListener('submit', function(e) {
                e.preventDefault();
                
                const email = document.getElementById('login-email').value;
                const password = document.getElementById('login-password').value;
                
                alert("Esta es una versión prototipo. La funcionalidad de inicio de sesión será implementada en la versión final.");
                closeModal('login-modal');
            });

            // Register form
            document.getElementById('register-form').addEventListener('submit', function(e) {
                e.preventDefault();
                
                const name = document.getElementById('register-name').value;
                const email = document.getElementById('register-email').value;
                const password = document.getElementById('register-password').value;
                const confirmPassword = document.getElementById('register-confirm-password').value;
                
                if (password !== confirmPassword) {
                    alert("Las contraseñas no coinciden.");
                    return;
                }
                
                alert("Esta es una versión prototipo. La funcionalidad de registro será implementada en la versión final.");
                closeModal('register-modal');
            });

            // Agent form
            document.getElementById('agent-form').addEventListener('submit', function(e) {
                e.preventDefault();
                
                const agentId = document.getElementById('agent-id').value;
                const agentPassword = document.getElementById('agent-password').value;
                
                alert("Esta es una versión prototipo. La funcionalidad de modo agente será implementada en la versión final.");
                closeModal('agent-modal');
            });

            // Theft report form
            document.getElementById('theft-report-form').addEventListener('submit', function(e) {
                e.preventDefault();
                
                const imei = document.getElementById('imei').value;
                const description = document.getElementById('description').value;
                
                alert("Esta es una versión prototipo. La funcionalidad de reporte de robo será implementada en la versión final.");
                closeModal('report-theft-modal');
            });
        }
    </script>
</body>
</html>
