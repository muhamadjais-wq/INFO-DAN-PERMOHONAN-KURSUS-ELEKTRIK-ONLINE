<!DOCTYPE html>
<html lang="ms-MY" class="scroll-smooth">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <!-- Viewport setting ini kritikal untuk Android & iPhone -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>JABATAN ELEKTRIK IKBN KINARUT SABAH - Program Kompetensi ECoS</title>
    <meta name="description" content="Portal rasmi maklumat dan permohonan kursus kompetensi elektrik IKBN Kinarut Sabah (PW2, PW4, A0-B0).">
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;700;800&family=Oswald:wght@500;700&display=swap" rel="stylesheet">

    <!-- Firebase SDKs (Kekal Sama) -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, runTransaction } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-ikbn-app';
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        if (Object.keys(firebaseConfig).length > 0) {
            const app = initializeApp(firebaseConfig);
            const auth = getAuth(app);
            const db = getFirestore(app);
            
            const initAuth = async () => {
                try {
                    if (initialAuthToken) {
                        await signInWithCustomToken(auth, initialAuthToken);
                    } else {
                        await signInAnonymously(auth);
                    }
                } catch (error) {
                    console.error("Auth Error:", error);
                    document.getElementById('visitor-count').innerText = "N/A";
                }
            };
            initAuth();

            onAuthStateChanged(auth, (user) => {
                if (user) handleVisitorCount(db);
            });
        } else {
            document.getElementById('visitor-count').innerText = "Demo";
        }

        async function handleVisitorCount(db) {
            const currentYear = new Date().getFullYear().toString();
            const visitorRef = doc(db, 'artifacts', appId, 'public', 'data', 'visitor_counters', currentYear);
            const lastVisitKey = 'ikbn_last_visit_date';
            const today = new Date().toDateString();
            const lastVisit = localStorage.getItem(lastVisitKey);

            try {
                if (lastVisit !== today) {
                    await runTransaction(db, async (transaction) => {
                        const sfDoc = await transaction.get(visitorRef);
                        let newCount = 1;
                        if (sfDoc.exists()) newCount = (sfDoc.data().count || 0) + 1;
                        transaction.set(visitorRef, { count: newCount }, { merge: true });
                    });
                    localStorage.setItem(lastVisitKey, today);
                }
                const docSnap = await getDoc(visitorRef);
                if (docSnap.exists()) animateValue("visitor-count", 0, docSnap.data().count, 1500);
            } catch (e) { console.error("Stats Error:", e); }
        }

        function animateValue(id, start, end, duration) {
            if (start === end) return;
            const range = end - start;
            let current = start;
            const increment = end > start ? 1 : -1;
            const stepTime = Math.abs(Math.floor(duration / range));
            const obj = document.getElementById(id);
            const timer = setInterval(function() {
                current += increment;
                obj.innerHTML = current.toLocaleString('ms-MY');
                if (current == end) clearInterval(timer);
            }, stepTime);
        }
    </script>

    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>

    <style>
        /* --- CORE STYLES --- */
        body { 
            font-family: 'Inter', sans-serif; 
            background-color: #f1f5f9; 
            background-image: radial-gradient(#cbd5e1 1px, transparent 1px);
            background-size: 24px 24px;
            -webkit-tap-highlight-color: transparent; /* Buang highlight biru pada Android/iOS bila tap */
        }
        h1, h2, h3, h4, .font-heading { font-family: 'Oswald', sans-serif; }
        
        :root { --ikbn-blue: #004d99; --ikbn-gold: #ffcc00; }
        .text-ikbn-blue { color: var(--ikbn-blue); }
        .bg-ikbn-blue { background-color: var(--ikbn-blue); }
        .text-ikbn-gold { color: var(--ikbn-gold); }
        .bg-ikbn-gold { background-color: var(--ikbn-gold); }

        /* --- ANIMATIONS --- */
        html { scroll-padding-top: 100px; }
        .fade-in-up { animation: fadeInUp 0.6s ease-out forwards; opacity: 0; transform: translateY(20px); }
        @keyframes fadeInUp { to { opacity: 1; transform: translateY(0); } }
        
        .electric-pulse { animation: pulseGlow 2s infinite; }
        @keyframes pulseGlow {
            0% { box-shadow: 0 0 0 0 rgba(255, 204, 0, 0.7); }
            70% { box-shadow: 0 0 0 10px rgba(255, 204, 0, 0); }
            100% { box-shadow: 0 0 0 0 rgba(255, 204, 0, 0); }
        }

        .flow-line { position: relative; overflow: hidden; }
        .flow-line::after {
            content: ''; position: absolute; top: 0; left: -100%; width: 100%; height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255,255,255,0.8), transparent);
            animation: flowLight 2s infinite;
        }
        @keyframes flowLight { 100% { left: 100%; } }

        /* --- NEW CHART ANIMATIONS (ELECTRON FLOW) --- */
        @keyframes travelRight {
            0% { left: 0; opacity: 0; }
            10% { opacity: 1; }
            90% { opacity: 1; }
            100% { left: 100%; opacity: 0; }
        }
        @keyframes travelDown {
            0% { top: 0; opacity: 0; }
            10% { opacity: 1; }
            90% { opacity: 1; }
            100% { top: 100%; opacity: 0; }
        }
        @keyframes nodePop {
            0% { transform: scale(1); }
            50% { transform: scale(1.1); box-shadow: 0 0 20px rgba(0,77,153,0.5); }
            100% { transform: scale(1); }
        }

        .electron-h { position: absolute; top: 50%; transform: translateY(-50%); width: 10px; height: 10px; background: #004d99; border-radius: 50%; box-shadow: 0 0 10px #00aaff; z-index: 10; animation: travelRight 2s linear infinite; }
        .electron-v { position: absolute; left: 50%; transform: translateX(-50%); width: 10px; height: 10px; background: #004d99; border-radius: 50%; box-shadow: 0 0 10px #00aaff; z-index: 10; animation: travelDown 2s linear infinite; }
        
        /* Node Animations Staggered */
        .node-animate { animation: nodePop 2s infinite; }
        .delay-0 { animation-delay: 0s; }
        .delay-1 { animation-delay: 0.5s; }
        .delay-2 { animation-delay: 1s; }
        .delay-3 { animation-delay: 1.5s; }
        .delay-4 { animation-delay: 2s; }
        .delay-5 { animation-delay: 2.5s; }

        .glass-panel { 
            background: rgba(15, 23, 42, 0.7); 
            backdrop-filter: blur(12px); 
            border: 1px solid rgba(255, 255, 255, 0.1); 
        }
        #hero-canvas { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; pointer-events: none; }
        
        /* Audio Pulse Animation */
        .audio-wave {
            display: inline-flex;
            align-items: center;
            justify-content: center;
            gap: 3px;
            height: 20px;
        }
        .audio-bar {
            width: 3px;
            background-color: #ffcc00;
            animation: audioBounce 1s infinite ease-in-out;
        }
        .audio-bar:nth-child(1) { height: 10px; animation-delay: 0s; }
        .audio-bar:nth-child(2) { height: 15px; animation-delay: 0.1s; }
        .audio-bar:nth-child(3) { height: 20px; animation-delay: 0.2s; }
        .audio-bar:nth-child(4) { height: 15px; animation-delay: 0.3s; }
        .audio-bar:nth-child(5) { height: 10px; animation-delay: 0.4s; }
        
        @keyframes audioBounce {
            0%, 100% { transform: scaleY(0.5); }
            50% { transform: scaleY(1); }
        }
    </style>
</head>
<body class="text-slate-800 antialiased relative">

    <!-- Scroll to Top -->
    <button id="scrollToTopBtn" class="fixed bottom-6 right-6 z-50 hidden bg-ikbn-gold text-ikbn-blue p-3 rounded-full shadow-xl hover:bg-yellow-400 hover:-translate-y-1 transition-all duration-300 border-2 border-ikbn-blue active:scale-90">
        <i data-lucide="chevron-up" class="w-6 h-6"></i>
    </button>

    <!-- SMART ASSISTANT FLOATING BUTTON -->
    <button onclick="openSmartModal()" class="fixed bottom-24 right-6 z-50 bg-ikbn-blue text-white p-4 rounded-full shadow-2xl hover:bg-blue-800 hover:scale-110 active:scale-95 transition-all duration-300 group border-4 border-white flex items-center gap-2">
        <i data-lucide="bot" class="w-6 h-6 animate-bounce"></i>
        <span class="font-bold text-sm hidden group-hover:block transition-all duration-300">Semak Kelayakan</span>
    </button>

    <!-- SMART ASSISTANT MODAL -->
    <div id="smart-modal" class="fixed inset-0 z-[60] hidden bg-black/60 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="bg-white rounded-2xl shadow-2xl max-w-md w-full overflow-hidden transform transition-all scale-95 opacity-0" id="smart-modal-content">
            <div class="bg-ikbn-blue p-6 text-white flex justify-between items-center">
                <div class="flex items-center gap-3">
                    <div class="bg-white/20 p-2 rounded-lg"><i data-lucide="bot" class="w-6 h-6 text-ikbn-gold"></i></div>
                    <h3 class="text-xl font-bold">Pembantu Pintar ECoS</h3>
                </div>
                <button onclick="closeSmartModal()" class="hover:text-red-300 active:scale-90 transition"><i data-lucide="x" class="w-6 h-6"></i></button>
            </div>
            
            <div class="p-6 max-h-[80vh] overflow-y-auto">
                <!-- Step 1: Current Status -->
                <div id="modal-step-1">
                    <p class="text-lg font-bold text-slate-700 mb-4">Pilih kelayakan tertinggi anda sekarang:</p>
                    <div class="space-y-3">
                        <button onclick="smartCheck('SPM')" class="w-full text-left p-3 rounded-xl border border-slate-200 hover:border-ikbn-blue hover:bg-blue-50 active:bg-blue-100 transition flex items-center gap-3 group">
                            <div class="bg-slate-100 p-2 rounded-full group-hover:bg-blue-200"><i data-lucide="book-open" class="w-5 h-5 text-slate-600 group-hover:text-blue-700"></i></div>
                            <div><span class="block font-bold text-slate-800">Tamat Tingkatan 5 / SPM</span><span class="text-xs text-slate-500">Tiada sijil kompetensi elektrik.</span></div>
                        </button>
                        <button onclick="smartCheck('PW2')" class="w-full text-left p-3 rounded-xl border border-slate-200 hover:border-ikbn-blue hover:bg-blue-50 active:bg-blue-100 transition flex items-center gap-3 group">
                            <div class="bg-slate-100 p-2 rounded-full group-hover:bg-blue-200"><i data-lucide="zap" class="w-5 h-5 text-slate-600 group-hover:text-blue-700"></i></div>
                            <div><span class="block font-bold text-slate-800">Ada PW1 / PW2</span><span class="text-xs text-slate-500">Pendawai Fasa Tunggal.</span></div>
                        </button>
                        <button onclick="smartCheck('A0')" class="w-full text-left p-3 rounded-xl border border-slate-200 hover:border-ikbn-blue hover:bg-blue-50 active:bg-blue-100 transition flex items-center gap-3 group">
                            <div class="bg-slate-100 p-2 rounded-full group-hover:bg-blue-200"><i data-lucide="zap" class="w-5 h-5 text-slate-600 group-hover:text-blue-700"></i></div>
                            <div><span class="block font-bold text-slate-800">Ada A0</span><span class="text-xs text-slate-500">Penjaga Jentera Asas.</span></div>
                        </button>
                        <button onclick="smartCheck('A1')" class="w-full text-left p-3 rounded-xl border border-slate-200 hover:border-ikbn-blue hover:bg-blue-50 active:bg-blue-100 transition flex items-center gap-3 group">
                            <div class="bg-slate-100 p-2 rounded-full group-hover:bg-blue-200"><i data-lucide="zap" class="w-5 h-5 text-slate-600 group-hover:text-blue-700"></i></div>
                            <div><span class="block font-bold text-slate-800">Ada A1</span><span class="text-xs text-slate-500">Talian Atas.</span></div>
                        </button>
                         <button onclick="smartCheck('A4')" class="w-full text-left p-3 rounded-xl border border-slate-200 hover:border-ikbn-blue hover:bg-blue-50 active:bg-blue-100 transition flex items-center gap-3 group">
                            <div class="bg-slate-100 p-2 rounded-full group-hover:bg-blue-200"><i data-lucide="zap" class="w-5 h-5 text-slate-600 group-hover:text-blue-700"></i></div>
                            <div><span class="block font-bold text-slate-800">Ada A4</span><span class="text-xs text-slate-500">Voltan Rendah Penuh.</span></div>
                        </button>
                    </div>
                </div>

                <!-- Result View -->
                <div id="modal-result" class="hidden text-center relative">
                    <!-- Audio Wave Animation (Hidden by default) -->
                    <div id="audio-visualizer" class="hidden absolute top-0 right-0">
                        <div class="audio-wave">
                            <div class="audio-bar"></div>
                            <div class="audio-bar"></div>
                            <div class="audio-bar"></div>
                            <div class="audio-bar"></div>
                            <div class="audio-bar"></div>
                        </div>
                    </div>

                    <div id="result-icon" class="mx-auto bg-green-100 w-16 h-16 rounded-full flex items-center justify-center mb-4 text-green-600 shadow-md">
                        <i data-lucide="check-circle" class="w-8 h-8"></i>
                    </div>
                    <h4 class="text-2xl font-black text-slate-800 mb-2">Cadangan Kami</h4>
                    <p class="text-slate-600 mb-6 text-sm leading-relaxed" id="result-text">...</p>
                    
                    <div class="bg-slate-50 p-4 rounded-xl border border-slate-200 mb-6 relative overflow-hidden">
                        <div class="absolute inset-0 bg-blue-50 opacity-50"></div>
                        <span class="relative z-10 text-xs font-bold text-slate-400 uppercase tracking-wider">Laluan Disyorkan</span>
                        <div class="relative z-10 text-2xl md:text-3xl font-black text-ikbn-blue mt-1 leading-tight" id="result-course-name">KURSUS</div>
                    </div>

                    <div class="flex gap-3">
                        <button onclick="speakCurrentText()" class="flex-none p-3 rounded-lg border border-slate-300 text-slate-600 hover:bg-blue-50 active:scale-95 transition" title="Ulang Suara">
                            <i data-lucide="volume-2" class="w-6 h-6"></i>
                        </button>
                        <button onclick="resetModal()" class="flex-1 py-3 rounded-lg border border-slate-300 text-slate-600 font-bold hover:bg-slate-50 active:scale-95 transition">Semula</button>
                        <a href="#kursus-ringkas" onclick="closeSmartModal()" class="flex-1 py-3 rounded-lg bg-ikbn-gold text-ikbn-blue font-bold hover:bg-yellow-400 active:scale-95 transition shadow-lg flex items-center justify-center">Lihat Kursus</a>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Navbar -->
    <header class="bg-ikbn-blue/95 backdrop-blur-md shadow-lg sticky top-0 z-40 w-full transition-all duration-300 border-b border-white/10">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex justify-between items-center h-20">
                <div class="flex items-center space-x-3 group cursor-pointer" onclick="window.scrollTo(0,0)">
                    <div class="bg-white p-1.5 rounded-full shadow-md group-hover:rotate-12 transition-transform duration-300">
                        <div class="h-8 w-8 flex items-center justify-center font-bold text-xs text-ikbn-blue border-2 border-ikbn-blue rounded-full">KBS</div>
                    </div>
                    <!-- TULISAN MERAH (text-red-400) UNTUK KONTRAS LEBIH BAIK PADA LATAR GELAP -->
                    <div class="text-red-400 leading-tight">
                        <h1 class="text-lg md:text-xl font-bold tracking-wide uppercase">Jabatan Elektrik</h1>
                        <p class="text-xs md:text-sm text-ikbn-gold font-medium tracking-wider">IKBN KINARUT SABAH</p>
                    </div>
                </div>
                
                <!-- MENU DESKTOP -->
                <nav class="hidden md:flex space-x-1">
                    <a href="#misi-visi" class="px-3 py-2 text-sm font-medium text-red-400 hover:text-ikbn-gold hover:bg-white/10 rounded-md transition">Visi & Misi</a>
                    <a href="#kursus-ringkas" class="px-3 py-2 text-sm font-medium text-red-400 hover:text-ikbn-gold hover:bg-white/10 rounded-md transition">Senarai Kursus</a>
                    <a href="#permohonan" class="px-3 py-2 text-sm font-medium text-red-400 hover:text-ikbn-gold hover:bg-white/10 rounded-md transition">Permohonan</a>
                    <a href="#hubungi" class="px-3 py-2 text-sm font-medium text-bg-ikbn-gold text-ikbn-blue bg-ikbn-gold rounded-md shadow-md hover:bg-yellow-300 active:scale-95 transition flex items-center animate-pulse hover:animate-none">
                        <i data-lucide="phone" class="w-4 h-4 mr-1.5"></i> Hubungi
                    </a>
                </nav>
                
                <!-- MENU MOBILE BUTTON -->
                <div class="md:hidden flex items-center">
                    <button id="mobile-menu-btn" class="text-red-400 hover:text-ikbn-gold p-2 focus:outline-none active:scale-90 transition">
                        <i data-lucide="menu" class="w-7 h-7"></i>
                    </button>
                </div>
            </div>
        </div>
        
        <!-- MENU MOBILE PANEL -->
        <div id="mobile-menu" class="md:hidden hidden bg-blue-900 border-t border-blue-800 absolute w-full shadow-xl">
            <div class="px-4 pt-2 pb-6 space-y-1">
                <a href="#misi-visi" class="mobile-link block px-3 py-3 text-base font-medium text-red-400 hover:bg-blue-800 rounded-md active:bg-blue-700">Visi & Misi</a>
                <a href="#kursus-ringkas" class="mobile-link block px-3 py-3 text-base font-medium text-red-400 hover:bg-blue-800 rounded-md active:bg-blue-700">Senarai Kursus</a>
                <a href="#permohonan" class="mobile-link block px-3 py-3 text-base font-medium text-red-400 hover:bg-blue-800 rounded-md active:bg-blue-700">Cara Memohon</a>
            </div>
        </div>
    </header>

    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8 space-y-16">

        <!-- Hero Section (Updated to Dark Theme for Contrast) -->
        <section class="relative bg-slate-900 rounded-2xl shadow-xl overflow-hidden border-t-4 border-ikbn-gold fade-in-up min-h-[500px] flex items-center" style="animation-delay: 0.1s;">
            <canvas id="hero-canvas"></canvas>
            <div class="p-8 md:p-12 relative z-10 flex flex-col md:flex-row items-center justify-between gap-10 w-full">
                <div class="flex-1 glass-panel p-8 rounded-2xl shadow-lg">
                    <div class="flex items-center gap-3 mb-4">
                        <div class="p-2 bg-slate-800 rounded-lg border border-slate-600"><i data-lucide="zap" class="w-8 h-8 text-ikbn-gold"></i></div>
                        <span class="text-sm font-bold tracking-widest text-blue-200 uppercase bg-slate-800/50 px-3 py-1 rounded-full border border-slate-600">Program ECoS</span>
                    </div>
                    <h2 class="text-4xl md:text-5xl font-black text-white mb-4 leading-tight">
                        Program Kompetensi <span class="text-ikbn-gold">Elektrik</span>
                    </h2>
                    <p class="text-lg text-slate-300 mb-6 max-w-2xl">
                        Mendaki tangga kerjaya elektrik anda dengan sijil yang diiktiraf Energy Commission of Sabah (ECoS). Daftar kursus Pendawai, Penjaga Jentera, atau Modular.
                    </p>
                    <div class="flex flex-wrap gap-4">
                        <a href="#kursus-ringkas" class="bg-ikbn-gold text-ikbn-blue px-6 py-3 rounded-lg font-bold shadow-lg hover:bg-yellow-400 active:scale-95 transition transform hover:-translate-y-1 flex items-center flow-line overflow-hidden">
                            <span class="relative z-10 flex items-center">Lihat Kursus <i data-lucide="arrow-down" class="ml-2 w-4 h-4"></i></span>
                        </a>
                        <button onclick="openSmartModal()" class="bg-transparent text-white border-2 border-white px-6 py-3 rounded-lg font-bold hover:bg-white/10 active:scale-95 transition transform hover:-translate-y-1 flex items-center">
                            <i data-lucide="bot" class="w-4 h-4 mr-2"></i> Bantuan Pintar
                        </button>
                    </div>
                    <div class="mt-4 flex items-center gap-2">
                        <span class="bg-green-900/50 text-green-200 text-xs font-bold px-2 py-1 rounded border border-green-700">HRDF Claimable</span>
                        <span class="bg-blue-900/50 text-blue-200 text-xs font-bold px-2 py-1 rounded border border-blue-700">ECoS Accredited</span>
                    </div>
                </div>
                
                <!-- Mandatory Warning Box -->
                <div class="w-full md:w-1/3 bg-white/90 backdrop-blur-sm border-l-4 border-red-600 p-6 rounded-r-lg shadow-xl">
                    <h3 class="flex items-center text-red-700 font-bold text-lg mb-3 border-b border-red-200 pb-2">
                        <i data-lucide="alert-triangle" class="w-5 h-5 mr-2"></i> SYARAT WAJIB ECoS
                    </h3>
                    <ul class="space-y-3 text-sm text-red-800 font-medium">
                        <li class="flex items-start"><i data-lucide="check-circle" class="w-4 h-4 mr-2 mt-0.5 shrink-0 text-green-600"></i> Calon & Majikan MESTI berdaftar dengan ECoS.</li>
                        <li class="flex items-start"><i data-lucide="alert-circle" class="w-4 h-4 mr-2 mt-0.5 shrink-0 text-orange-500"></i> Pemegang Sijil ST (Suruhanjaya Tenaga) perlu setara (daftar semula) dengan ECoS.</li>
                        <li class="flex items-start"><i data-lucide="user" class="w-4 h-4 mr-2 mt-0.5 shrink-0 text-blue-600"></i> Warganegara Malaysia shj (Tingkatan 5 / SPM).</li>
                    </ul>
                </div>
            </div>
        </section>

        <!-- Stats & Social Links -->
        <section class="grid grid-cols-1 md:grid-cols-4 gap-6 fade-in-up" style="animation-delay: 0.2s;">
            
            <!-- VISITOR COUNTER CARD (NEW ANIMATED DESIGN) -->
            <div class="group relative bg-slate-900 p-0.5 rounded-2xl shadow-2xl hover:-translate-y-1 transition duration-500">
                <!-- Glowing Border Effect -->
                <div class="absolute inset-0 bg-gradient-to-r from-blue-500 via-yellow-400 to-blue-500 opacity-75 blur-sm rounded-2xl animate-pulse group-hover:opacity-100 transition duration-500"></div>
                
                <!-- Card Content -->
                <div class="relative bg-slate-900 h-full p-6 rounded-2xl flex flex-col justify-between overflow-hidden">
                    <!-- Decorative Background Icon -->
                    <div class="absolute -right-6 -top-6 text-slate-800 opacity-50 transform rotate-12 group-hover:scale-110 transition duration-700">
                        <i data-lucide="bar-chart-2" class="w-32 h-32"></i>
                    </div>

                    <!-- Header -->
                    <div class="relative z-10 flex items-center space-x-3 mb-2">
                        <div class="p-2 bg-blue-600/30 rounded-lg border border-blue-500/50 shadow-[0_0_15px_rgba(59,130,246,0.5)]">
                            <i data-lucide="users" class="w-6 h-6 text-blue-300"></i>
                        </div>
                        <span class="text-sm font-bold uppercase tracking-wider text-blue-100 drop-shadow-md">Jumlah Pelawat</span>
                    </div>

                    <!-- Counter Number -->
                    <div class="relative z-10 text-5xl md:text-6xl font-black text-white tracking-tighter drop-shadow-[0_2px_10px_rgba(0,0,0,0.5)]" id="visitor-count">
                        ...
                    </div>
                    
                    <!-- Live Indicator -->
                    <div class="relative z-10 mt-4 pt-4 border-t border-slate-700 flex items-center">
                        <span class="relative flex h-3 w-3 mr-2">
                          <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-green-400 opacity-75"></span>
                          <span class="relative inline-flex rounded-full h-3 w-3 bg-green-500"></span>
                        </span>
                        <span class="text-xs font-mono text-green-400 tracking-widest">ONLINE</span>
                    </div>
                </div>
            </div>

            <div class="md:col-span-3 bg-white p-6 rounded-2xl shadow-lg border border-slate-100">
                <h3 class="text-lg font-bold text-slate-700 mb-4 flex items-center border-b pb-2"><i data-lucide="share-2" class="w-5 h-5 mr-2 text-ikbn-blue"></i> Pautan Pantas</h3>
                <div class="grid grid-cols-2 sm:grid-cols-4 gap-4">
                    <a href="https://forms.gle/uvJa9qgGkYsAcFVr7" target="_blank" class="flex flex-col items-center justify-center p-4 rounded-xl bg-slate-50 hover:bg-blue-50 active:bg-blue-100 transition border border-slate-200"><i data-lucide="message-square" class="w-8 h-8 text-blue-600 mb-2"></i><span class="text-xs font-bold text-slate-600">MAKLUM BALAS</span></a>
                    <a href="https://www.facebook.com/people/Jabatan-Elektrik-Ikbn-Kinarut/100073449556010/" target="_blank" class="flex flex-col items-center justify-center p-4 rounded-xl bg-slate-50 hover:bg-blue-50 active:bg-blue-100 transition border border-slate-200"><i data-lucide="facebook" class="w-8 h-8 text-blue-800 mb-2"></i><span class="text-xs font-bold text-slate-600">FACEBOOK</span></a>
                    <a href="https://www.tiktok.com/@jabatan.elektrik" target="_blank" class="flex flex-col items-center justify-center p-4 rounded-xl bg-slate-50 hover:bg-slate-100 active:bg-slate-200 transition border border-slate-200"><span class="w-8 h-8 flex items-center justify-center mb-2"><svg viewBox="0 0 24 24" fill="currentColor" class="w-7 h-7 text-black"><path d="M19.59 6.69a4.83 4.83 0 0 1-3.77-4.25V2h-3.45v13.67a2.89 2.89 0 0 1-5.2 1.74 2.89 2.89 0 0 1 2.31-4.64 2.93 2.93 0 0 1 .88.13V9.4a6.84 6.84 0 0 0-1-.05A6.33 6.33 0 0 0 5 20.1a6.34 6.34 0 0 0 10.86-4.43v-7a8.16 8.16 0 0 0 4.77 1.52v-3.4a4.85 4.85 0 0 1-1-.1z"/></svg></span><span class="text-xs font-bold text-slate-600">TIKTOK</span></a>
                    <a href="https://maps.app.goo.gl/UbdhERujykyLpUrJ7" target="_blank" class="flex flex-col items-center justify-center p-4 rounded-xl bg-slate-50 hover:bg-red-50 active:bg-red-100 transition border border-slate-200"><i data-lucide="map-pin" class="w-8 h-8 text-red-600 mb-2"></i><span class="text-xs font-bold text-slate-600">LOKASI</span></a>
                </div>
            </div>
        </section>

        <!-- Course Listings -->
        <section id="kursus-ringkas" class="space-y-8">
            <div class="text-center max-w-3xl mx-auto mb-12">
                <span class="text-ikbn-blue font-bold tracking-wider text-sm uppercase">Pilihan Program</span>
                <h2 class="text-3xl md:text-4xl font-black text-slate-900 mt-2">Senarai Kursus & Modul</h2>
                <div class="h-1 w-20 bg-ikbn-gold mx-auto mt-4 rounded-full"></div>
            </div>

            <!-- KURSUS PENDAWAI (PW) -->
            <h3 class="text-2xl font-bold text-slate-800 border-l-4 border-slate-600 pl-4 mb-4">Kategori Pendawai (Wireman)</h3>
            <div class="grid md:grid-cols-2 lg:grid-cols-2 gap-8 mb-12">
                <!-- PW2 -->
                <article class="flex flex-col bg-white rounded-2xl shadow-lg border border-slate-200 hover:shadow-2xl transition duration-300">
                    <div class="bg-slate-700 p-6 text-white relative"><h3 class="text-3xl font-bold">PW2</h3><p class="text-slate-300 text-sm mt-1">Pendawai Fasa Tunggal</p></div>
                    <div class="p-6 flex-grow space-y-4">
                        <div class="grid grid-cols-2 gap-4 text-sm border-b pb-4"><div class="font-bold">RM2,900</div><div class="text-right">4 Bulan (240 Jam)</div></div>
                        <ul class="text-sm text-slate-600 space-y-1 list-disc ml-4">
                            <li>Umur min 18 Tahun, Tamat Tingkatan 5.</li>
                            <li><strong>2 Tahun</strong> pengalaman dengan kontraktor ECoS.</li>
                            <li>Wajib ada Buku Log.</li>
                        </ul>
                    </div>
                    <!-- UPDATE BUTTON PW2 -->
                    <div class="p-4 bg-slate-50 border-t"><a href="#hubungi" class="block w-full text-center bg-ikbn-gold text-ikbn-blue font-bold py-3 rounded-lg hover:bg-yellow-400 active:scale-95 transition shadow-md">Hubungi Urusetia</a></div>
                </article>
                <!-- PW4 -->
                <article class="flex flex-col bg-white rounded-2xl shadow-lg border border-slate-200 hover:shadow-2xl transition duration-300">
                    <div class="bg-slate-700 p-6 text-white relative"><h3 class="text-3xl font-bold">PW4</h3><p class="text-slate-300 text-sm mt-1">Pendawai Fasa Tiga</p></div>
                    <div class="p-6 flex-grow space-y-4">
                        <div class="grid grid-cols-2 gap-4 text-sm border-b pb-4"><div class="font-bold">RM2,900</div><div class="text-right">4 Bulan (240 Jam)</div></div>
                        <ul class="text-sm text-slate-600 space-y-1 list-disc ml-4">
                            <li>Umur min 18 Tahun.</li>
                            <li>Ada sijil PW1/PW2 min 1 Tahun.</li>
                            <li><strong>1 Tahun</strong> pengalaman fasa 3 dengan kontraktor Kelas C ke atas.</li>
                        </ul>
                    </div>
                    <!-- UPDATE BUTTON PW4 -->
                    <div class="p-4 bg-slate-50 border-t"><a href="#hubungi" class="block w-full text-center bg-ikbn-gold text-ikbn-blue font-bold py-3 rounded-lg hover:bg-yellow-400 active:scale-95 transition shadow-md">Hubungi Urusetia</a></div>
                </article>
            </div>

            <!-- KURSUS PENJAGA JENTERA (CHARGEMAN) -->
            <h3 class="text-2xl font-bold text-slate-800 border-l-4 border-ikbn-blue pl-4 mb-4">Kategori Penjaga Jentera (Chargeman)</h3>
            <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-8 mb-12">
                <!-- A0 -->
                <article class="flex flex-col bg-white rounded-2xl shadow-lg border border-slate-200 hover:shadow-2xl transition duration-300 h-full">
                    <div class="bg-ikbn-blue p-6 text-white relative overflow-hidden">
                        <div class="absolute top-0 right-0 p-4 opacity-20"><i data-lucide="zap" class="w-16 h-16"></i></div>
                        <h3 class="text-3xl font-bold relative z-10">A0</h3>
                        <p class="text-blue-200 text-sm mt-1 relative z-10">Sistem Voltan Rendah</p>
                    </div>
                    <div class="p-6 flex-grow space-y-4">
                        <div class="grid grid-cols-2 gap-4 text-sm border-b pb-4"><div class="font-bold">RM4,100</div><div class="text-right">6 Bulan (480 Jam)</div></div>
                        <ul class="text-sm text-slate-600 space-y-1 list-disc ml-4">
                            <li>Umur min 20 Tahun, Tamat SPM.</li>
                            <li><strong>3 Tahun</strong> pengalaman elektrik hidup.</li>
                            <li>ATAU <strong>2 Tahun</strong> jika ada sijil PW1-PW6/Politeknik.</li>
                        </ul>
                    </div>
                    <div class="p-4 bg-slate-50 border-t"><a href="https://forms.gle/ZeXBrrAGaYtxr2u67" target="_blank" class="block w-full text-center bg-ikbn-gold text-ikbn-blue font-bold py-3 rounded-lg hover:bg-yellow-400 active:scale-95 transition shadow-md">Mohon A0</a></div>
                </article>

                <!-- A1 -->
                <article class="flex flex-col bg-white rounded-2xl shadow-lg border border-slate-200 hover:shadow-2xl transition duration-300 h-full">
                    <div class="bg-ikbn-blue p-6 text-white relative overflow-hidden">
                        <div class="absolute top-0 right-0 p-4 opacity-20"><i data-lucide="zap" class="w-16 h-16"></i></div>
                        <h3 class="text-3xl font-bold relative z-10">A1</h3>
                        <p class="text-blue-200 text-sm mt-1 relative z-10">VR + Talian Atas</p>
                    </div>
                    <div class="p-6 flex-grow space-y-4">
                        <div class="grid grid-cols-2 gap-4 text-sm border-b pb-4"><div class="font-bold">RM4,500</div><div class="text-right">6 Bulan (540 Jam)</div></div>
                        <ul class="text-sm text-slate-600 space-y-1 list-disc ml-4">
                            <li><strong>3 Tahun</strong> pengalaman (termasuk 1 tahun Talian Atas).</li>
                            <li>ATAU A0 + 1 tahun Talian Atas.</li>
                            <li>ATAU PW + 2 tahun (1 thn hidup + 1 thn talian atas).</li>
                        </ul>
                    </div>
                    <div class="p-4 bg-slate-50 border-t"><a href="https://forms.gle/TJpNuYjMt9GUGNHG8" target="_blank" class="block w-full text-center bg-ikbn-gold text-ikbn-blue font-bold py-3 rounded-lg hover:bg-yellow-400 active:scale-95 transition shadow-md">Mohon A1</a></div>
                </article>

                <!-- A4 -->
                <article class="flex flex-col bg-white rounded-2xl shadow-lg border border-slate-200 hover:shadow-2xl transition duration-300 h-full">
                    <div class="bg-ikbn-blue p-6 text-white relative overflow-hidden">
                        <div class="absolute top-0 right-0 p-4 opacity-20"><i data-lucide="zap" class="w-16 h-16"></i></div>
                        <h3 class="text-3xl font-bold relative z-10">A4</h3>
                        <p class="text-blue-200 text-sm mt-1 relative z-10">VR + Janakuasa</p>
                    </div>
                    <div class="p-6 flex-grow space-y-4">
                        <div class="grid grid-cols-2 gap-4 text-sm border-b pb-4"><div class="font-bold">RM5,000</div><div class="text-right">10 Bulan (600 Jam)</div></div>
                        <ul class="text-sm text-slate-600 space-y-1 list-disc ml-4">
                            <li>A0 + 1 thn Genset + 1 thn Overhead.</li>
                            <li>ATAU A1 + 1 thn Genset.</li>
                            <li>Kurang pengalaman? Ambil Modul TAVR/JKVR.</li>
                        </ul>
                    </div>
                    <div class="p-4 bg-slate-50 border-t"><a href="https://forms.gle/28Fy1RD8dZukcUrP6" target="_blank" class="block w-full text-center bg-ikbn-gold text-ikbn-blue font-bold py-3 rounded-lg hover:bg-yellow-400 active:scale-95 transition shadow-md">Mohon A4</a></div>
                </article>

                <!-- B0 -->
                <article class="flex flex-col bg-slate-800 rounded-2xl shadow-xl border border-slate-700 hover:shadow-2xl transition duration-300 h-full relative lg:col-span-3">
                    <div class="absolute top-0 right-0 bg-red-600 text-white text-xs font-bold px-3 py-1 rounded-bl-lg z-10">HIGH VOLTAGE</div>
                    <div class="bg-gradient-to-r from-slate-900 to-slate-800 p-6 text-white border-b border-slate-600 relative overflow-hidden">
                        <div class="absolute top-0 right-0 p-4 opacity-10"><i data-lucide="zap" class="w-16 h-16"></i></div>
                        <h3 class="text-3xl font-bold text-ikbn-gold relative z-10">B0 (11kV)</h3>
                        <p class="text-slate-300 text-sm mt-1 relative z-10">Voltan Tinggi</p>
                    </div>
                    <div class="p-6 grid md:grid-cols-2 gap-6 text-slate-200">
                        <div class="space-y-2">
                            <div class="flex justify-between border-b border-slate-600 pb-2"><span>Yuran</span><span class="font-bold text-white">RM18,700</span></div>
                            <div class="flex justify-between border-b border-slate-600 pb-2"><span>Tempoh</span><span class="font-bold text-white">10 Bulan (600 Jam)</span></div>
                            <div class="flex justify-between border-b border-slate-600 pb-2"><span>Kehadiran</span><span class="font-bold text-red-400">Wajib 95%</span></div>
                        </div>
                        <div>
                            <h4 class="font-bold text-ikbn-gold mb-2">Syarat Kelayakan:</h4>
                            <ul class="text-sm text-slate-300 space-y-1 list-disc ml-4">
                                <li>Ada perakuan <strong>A4 + 2 Tahun</strong> pengalaman Voltan Tinggi.</li>
                                <li>ATAU B0-1 + 1 Tahun pengalaman Genset VR.</li>
                                <li>Buku log HV lengkap.</li>
                            </ul>
                            <a href="https://forms.gle/zaMeYgkWMUbQ1HP18" target="_blank" class="mt-4 block w-full text-center bg-red-600 hover:bg-red-700 text-white font-bold py-2 rounded-lg active:scale-95 transition shadow-md">Mohon B0</a>
                        </div>
                    </div>
                </article>
            </div>

            <!-- MODUL & KOMPETENSI TERHAD -->
            <h3 class="text-2xl font-bold text-slate-800 border-l-4 border-slate-400 pl-4 mb-4">Kursus Modular (Pengganti Pengalaman)</h3>
            <div class="grid md:grid-cols-2 lg:grid-cols-4 gap-4">
                <!-- Modul TAVR -->
                <div class="bg-white p-4 rounded-xl shadow border border-slate-200 hover:shadow-md transition flex flex-col h-full">
                    <div class="flex-grow">
                        <h4 class="font-bold text-slate-800">Modul TAVR</h4>
                        <p class="text-xs text-slate-500 mb-2">Talian Atas Voltan Rendah</p>
                        <p class="text-sm font-bold text-ikbn-blue">RM2,000 / 80 Jam</p>
                        <p class="text-xs mt-2 text-slate-600">Untuk pemegang A0 (2 thn) ganti pengalaman ke A4.</p>
                    </div>
                    <a href="https://forms.gle/wtq2pc6yYB7UjzM59" target="_blank" class="mt-3 block w-full text-center bg-slate-700 hover:bg-slate-800 text-white text-xs font-bold py-2 rounded active:scale-95 transition">Mohon TAVR</a>
                </div>

                <!-- Modul JKVR -->
                <div class="bg-white p-4 rounded-xl shadow border border-slate-200 hover:shadow-md transition flex flex-col h-full">
                    <div class="flex-grow">
                        <h4 class="font-bold text-slate-800">Modul JKVR</h4>
                        <p class="text-xs text-slate-500 mb-2">Janakuasa Voltan Rendah</p>
                        <p class="text-sm font-bold text-ikbn-blue">RM2,000 / 80 Jam</p>
                        <p class="text-xs mt-2 text-slate-600">Untuk A0/A1 ganti pengalaman genset ke A4.</p>
                    </div>
                    <a href="https://forms.gle/acb7zLHTXScLzYoJ7" target="_blank" class="mt-3 block w-full text-center bg-slate-700 hover:bg-slate-800 text-white text-xs font-bold py-2 rounded active:scale-95 transition">Mohon JKVR</a>
                </div>

                <!-- Modul Pencawang (MKP) -->
                <div class="bg-white p-4 rounded-xl shadow border border-slate-200 hover:shadow-md transition flex flex-col h-full">
                    <div class="flex-grow">
                        <h4 class="font-bold text-slate-800">Modul Pencawang (MKP)</h4>
                        <p class="text-xs text-slate-500 mb-2">Kendalian Pencawang 11kV</p>
                        <p class="text-sm font-bold text-ikbn-blue">RM4,800 / 80 Jam</p>
                        <p class="text-xs mt-2 text-slate-600">Untuk B0 Terhad (Syarat: Kompeten VR + 3 thn kerja).</p>
                    </div>
                    <a href="https://forms.gle/3cAfRxsn9AQe63Mv5" target="_blank" class="mt-3 block w-full text-center bg-slate-700 hover:bg-slate-800 text-white text-xs font-bold py-2 rounded active:scale-95 transition">Mohon MKP</a>
                </div>

                <!-- Modul Enakmen -->
                <div class="bg-white p-4 rounded-xl shadow border border-slate-200 hover:shadow-md transition flex flex-col h-full">
                    <div class="flex-grow">
                        <h4 class="font-bold text-slate-800">Modul Enakmen</h4>
                        <p class="text-xs text-slate-500 mb-2">Enakmen Elektrik 2024</p>
                        <p class="text-sm font-bold text-ikbn-blue">RM185</p>
                        <p class="text-xs mt-2 text-slate-600">Wajib untuk pengiktirafan perundangan Sabah.</p>
                    </div>
                    <a href="#hubungi" class="mt-3 block w-full text-center border border-slate-300 text-slate-600 text-xs font-bold py-2 rounded hover:bg-slate-100 active:scale-95 transition">Hubungi Urusetia</a>
                </div>
            </div>
        </section>

        <!-- Animated Progression Flowchart (UPDATED FOR MOBILE) -->
        <section id="progression" class="bg-white rounded-2xl shadow-lg p-8 relative overflow-hidden">
            <div class="text-center mb-10 relative z-10">
                <h2 class="text-3xl font-bold text-slate-800">Laluan Kerjaya</h2>
                <p class="text-slate-500">Mendaki tangga kompetensi dari asas ke pakar. (Klik ikon untuk info & suara)</p>
            </div>
            
            <div class="flex flex-col md:flex-row justify-center items-center space-y-4 md:space-y-0 overflow-x-auto pb-4 relative z-10">
                
                <!-- NODE 1: SPM -->
                <div class="flex flex-col items-center group cursor-pointer node-animate delay-0" onclick="smartCheck('SPM'); openSmartModal();">
                    <div class="w-24 h-24 rounded-full border-4 border-slate-300 flex items-center justify-center bg-slate-50 group-hover:bg-slate-200 active:scale-95 transition duration-300 z-20 relative">
                        <span class="text-lg font-black text-slate-500">SPM</span>
                    </div>
                </div>

                <!-- PATH 1 -->
                <div class="hidden md:flex items-center relative w-16 h-2 mx-2">
                    <div class="w-full h-1 bg-slate-200 rounded"></div>
                    <div class="electron-h delay-1"></div>
                </div>
                <div class="md:hidden flex items-center relative h-16 w-2 my-2">
                    <div class="h-full w-1 bg-slate-200 rounded"></div>
                    <div class="electron-v delay-1"></div>
                </div>
                
                <!-- NODE 2: PW -->
                <div class="flex flex-col items-center group cursor-pointer node-animate delay-1" onclick="smartCheck('PW2'); openSmartModal();">
                    <div class="w-24 h-24 rounded-full border-4 border-slate-600 flex items-center justify-center bg-slate-50 group-hover:bg-slate-600 active:scale-95 transition duration-300 z-20 relative">
                        <span class="text-lg font-black text-slate-600 group-hover:text-white">PW</span>
                    </div>
                </div>

                <!-- PATH 2 -->
                <div class="hidden md:flex items-center relative w-16 h-2 mx-2">
                    <div class="w-full h-1 bg-slate-200 rounded"></div>
                    <div class="electron-h delay-2"></div>
                </div>
                <div class="md:hidden flex items-center relative h-16 w-2 my-2">
                    <div class="h-full w-1 bg-slate-200 rounded"></div>
                    <div class="electron-v delay-2"></div>
                </div>

                <!-- NODE 3: A0 -->
                <div class="flex flex-col items-center group cursor-pointer node-animate delay-2" onclick="smartCheck('A0'); openSmartModal();">
                    <div class="w-24 h-24 rounded-full border-4 border-ikbn-blue flex items-center justify-center bg-blue-50 group-hover:bg-ikbn-blue active:scale-95 transition duration-300 z-20 relative">
                        <span class="text-lg font-black text-ikbn-blue group-hover:text-white">A0</span>
                    </div>
                </div>

                <!-- PATH 3 -->
                <div class="hidden md:flex items-center relative w-16 h-2 mx-2">
                    <div class="w-full h-1 bg-slate-200 rounded"></div>
                    <div class="electron-h delay-3"></div>
                </div>
                <div class="md:hidden flex items-center relative h-16 w-2 my-2">
                    <div class="h-full w-1 bg-slate-200 rounded"></div>
                    <div class="electron-v delay-3"></div>
                </div>

                <!-- NODE 4: A1 -->
                <div class="flex flex-col items-center group cursor-pointer node-animate delay-3" onclick="smartCheck('A1'); openSmartModal();">
                    <div class="w-24 h-24 rounded-full border-4 border-ikbn-gold flex items-center justify-center bg-yellow-50 group-hover:bg-ikbn-gold active:scale-95 transition duration-300 z-20 relative">
                        <span class="text-lg font-black text-slate-800 group-hover:text-white">A1</span>
                    </div>
                </div>

                <!-- PATH 4 -->
                <div class="hidden md:flex items-center relative w-16 h-2 mx-2">
                    <div class="w-full h-1 bg-slate-200 rounded"></div>
                    <div class="electron-h delay-4"></div>
                </div>
                <div class="md:hidden flex items-center relative h-16 w-2 my-2">
                    <div class="h-full w-1 bg-slate-200 rounded"></div>
                    <div class="electron-v delay-4"></div>
                </div>

                <!-- NODE 5: A4 -->
                <div class="flex flex-col items-center group cursor-pointer node-animate delay-4" onclick="smartCheck('A4'); openSmartModal();">
                    <div class="w-24 h-24 rounded-full border-4 border-slate-800 flex items-center justify-center bg-slate-100 group-hover:bg-slate-800 active:scale-95 transition duration-300 z-20 relative">
                        <span class="text-lg font-black text-slate-800 group-hover:text-white">A4</span>
                    </div>
                </div>

                <!-- PATH 5 -->
                <div class="hidden md:flex items-center relative w-16 h-2 mx-2">
                    <div class="w-full h-1 bg-slate-200 rounded"></div>
                    <div class="electron-h delay-5" style="background: #dc2626; box-shadow: 0 0 10px #f87171;"></div>
                </div>
                <div class="md:hidden flex items-center relative h-16 w-2 my-2">
                    <div class="h-full w-1 bg-slate-200 rounded"></div>
                    <div class="electron-v delay-5" style="background: #dc2626; box-shadow: 0 0 10px #f87171;"></div>
                </div>

                 <!-- NODE 6: B0 -->
                 <div class="flex flex-col items-center group node-animate delay-5" onclick="smartCheck('B0'); openSmartModal();">
                    <div class="w-24 h-24 rounded-full border-4 border-red-600 flex items-center justify-center bg-red-50 group-hover:bg-red-600 active:scale-95 transition duration-300 shadow-xl z-20 relative cursor-pointer">
                        <span class="text-lg font-black text-red-600 group-hover:text-white">B0</span>
                    </div>
                </div>
            </div>
        </section>

    </main>

    <!-- Footer & Contact -->
    <footer id="hubungi" class="bg-slate-900 text-white mt-20 border-t-8 border-ikbn-gold">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
            <div class="grid md:grid-cols-2 lg:grid-cols-3 gap-8 text-center md:text-left">
                <!-- Column 1: Intro -->
                <div>
                    <h3 class="text-2xl font-bold text-ikbn-gold mb-2">HUBUNGI KAMI</h3>
                    <p class="text-slate-400 text-sm mb-4">Urusetia Kursus Separuh Masa & Modular Jabatan Elektrik IKBN Kinarut.</p>
                    <div class="flex flex-col space-y-2 text-sm text-slate-300">
                        <p class="font-bold text-white"><i data-lucide="home" class="w-4 h-4 inline mr-1"></i> Penginapan (Homestay):</p>
                        <p>Puan Queen: 011-6486 5345</p>
                    </div>
                </div>
                
                <!-- Column 2: Pegawai -->
                <div class="space-y-3">
                    <p class="font-bold text-ikbn-gold">URUSETIA PENJAGA JENTERA ELEKTRIK (PJE):</p>
                    <div class="flex items-center md:justify-start justify-center space-x-2">
                        <i data-lucide="phone" class="w-4 h-4 text-slate-400"></i>
                        <span>En. Muhamad Jais: 017-898 6747</span>
                    </div>
                    <div class="flex items-center md:justify-start justify-center space-x-2">
                        <i data-lucide="phone" class="w-4 h-4 text-slate-400"></i>
                        <span>En. Norazmi Bin Ali: 019-580 7494</span>
                    </div>
                    <p class="font-bold text-ikbn-gold mt-4">URUSETIA PENGAMBILAN KURSUS PENDAWAI ELEKTRIK (PW):</p>
                    <div class="flex items-center md:justify-start justify-center space-x-2">
                        <i data-lucide="phone" class="w-4 h-4 text-slate-400"></i>
                        <span>En. Hanif: 012-511 9108</span>
                    </div>
                    <div class="flex items-center md:justify-start justify-center space-x-2">
                        <i data-lucide="phone" class="w-4 h-4 text-slate-400"></i>
                        <span>En. Asyirul: 016-630 6121</span>
                    </div>
                </div>

                <!-- Column 3: Lokasi -->
                <div class="flex flex-col items-center md:items-start">
                    <a href="https://maps.app.goo.gl/UbdhERujykyLpUrJ7" target="_blank" class="flex items-center justify-center md:justify-start space-x-2 text-slate-300 hover:text-white active:bg-slate-800 transition bg-slate-800 px-4 py-2 rounded-lg mb-4">
                        <i data-lucide="map" class="w-5 h-5"></i><span>Peta Kampus (Google Maps)</span>
                    </a>
                    <p class="text-slate-500 text-xs">&copy; 2024 Jabatan Elektrik IKBN Kinarut.</p>
                    <p class="text-slate-600 text-xs mt-1">Kehadiran kursus WAJIB 90-95%.</p>
                </div>
            </div>
        </div>
    </footer>

    <!-- Scripts Logic -->
    <script>
        // Init Icons
        lucide.createIcons();

        // Mobile Menu
        const btn = document.getElementById('mobile-menu-btn');
        const menu = document.getElementById('mobile-menu');
        btn.addEventListener('click', () => { menu.classList.toggle('hidden'); });
        document.querySelectorAll('.mobile-link').forEach(link => {
            link.addEventListener('click', () => { menu.classList.add('hidden'); });
        });

        // Scroll To Top
        const scrollToTopBtn = document.getElementById('scrollToTopBtn');
        window.onscroll = function() {
            if (document.body.scrollTop > 300 || document.documentElement.scrollTop > 300) {
                scrollToTopBtn.classList.remove('hidden');
            } else {
                scrollToTopBtn.classList.add('hidden');
            }
        };
        scrollToTopBtn.addEventListener('click', () => {
            window.scrollTo({ top: 0, behavior: 'smooth' });
        });

        /* --- HERO CANVAS SIMULATION --- */
        const canvas = document.getElementById('hero-canvas');
        const ctx = canvas.getContext('2d');
        let width, height;
        let particles = [];

        function resize() {
            width = canvas.width = canvas.offsetWidth;
            height = canvas.height = canvas.offsetHeight;
        }

        class Particle {
            constructor() {
                this.x = Math.random() * width;
                this.y = Math.random() * height;
                this.vx = (Math.random() - 0.5) * 0.5;
                this.vy = (Math.random() - 0.5) * 0.5;
                this.size = Math.random() * 2 + 1;
            }
            update() {
                this.x += this.vx;
                this.y += this.vy;
                if (this.x < 0 || this.x > width) this.vx *= -1;
                if (this.y < 0 || this.y > height) this.vy *= -1;
            }
            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(56, 189, 248, 0.5)'; // Light Blue/Cyan for dark bg
                ctx.fill();
            }
        }

        function initParticles() {
            particles = [];
            const count = Math.min(width / 10, 80); 
            for (let i = 0; i < count; i++) {
                particles.push(new Particle());
            }
        }

        function animateCanvas() {
            ctx.clearRect(0, 0, width, height);
            for (let i = 0; i < particles.length; i++) {
                particles[i].update();
                particles[i].draw();
                for (let j = i; j < particles.length; j++) {
                    const dx = particles[i].x - particles[j].x;
                    const dy = particles[i].y - particles[j].y;
                    const dist = Math.sqrt(dx*dx + dy*dy);
                    if (dist < 100) {
                        ctx.beginPath();
                        ctx.strokeStyle = `rgba(56, 189, 248, ${0.15 - dist/1000})`; // Cyan lines
                        ctx.lineWidth = 0.5;
                        ctx.moveTo(particles[i].x, particles[i].y);
                        ctx.lineTo(particles[j].x, particles[j].y);
                        ctx.stroke();
                    }
                }
            }
            requestAnimationFrame(animateCanvas);
        }

        window.addEventListener('resize', () => { resize(); initParticles(); });
        resize();
        initParticles();
        animateCanvas();

        /* --- TTS & SMART ASSISTANT LOGIC --- */
        const smartModal = document.getElementById('smart-modal');
        const modalContent = document.getElementById('smart-modal-content');
        const modalStep1 = document.getElementById('modal-step-1');
        const modalResult = document.getElementById('modal-result');
        const resultText = document.getElementById('result-text');
        const resultCourse = document.getElementById('result-course-name');
        const audioVisualizer = document.getElementById('audio-visualizer');
        
        let currentUtterance = null;

        // Init Voices
        let voices = [];
        function loadVoices() {
            voices = window.speechSynthesis.getVoices();
        }
        window.speechSynthesis.onvoiceschanged = loadVoices;
        loadVoices(); // Init load

        function speakText(text) {
            window.speechSynthesis.cancel(); // Stop any current speech
            
            const utterance = new SpeechSynthesisUtterance(text);
            utterance.lang = 'ms-MY'; // Default Malay
            utterance.rate = 1.0;
            utterance.pitch = 1.1; // Higher pitch for female-like quality

            // Try to find a specific Malay/Indonesian female voice if possible
            // Note: Browser support varies wildly. We prefer MS-MY or ID-ID.
            const preferredVoice = voices.find(v => 
                (v.lang === 'ms-MY' || v.lang === 'id-ID') && 
                (v.name.includes('Google') || v.name.includes('Female'))
            ) || voices.find(v => v.lang === 'ms-MY') || voices.find(v => v.lang === 'id-ID');

            if (preferredVoice) {
                utterance.voice = preferredVoice;
            }

            // Visualizer Control
            utterance.onstart = () => {
                audioVisualizer.classList.remove('hidden');
            };
            utterance.onend = () => {
                audioVisualizer.classList.add('hidden');
            };
            utterance.onerror = () => {
                audioVisualizer.classList.add('hidden');
            };

            currentUtterance = utterance;
            window.speechSynthesis.speak(utterance);
        }

        function speakCurrentText() {
            if (resultText.innerText) {
                speakText(resultText.innerText);
            }
        }

        function openSmartModal() {
            smartModal.classList.remove('hidden');
            setTimeout(() => {
                modalContent.classList.remove('scale-95', 'opacity-0');
                modalContent.classList.add('scale-100', 'opacity-100');
            }, 10);
        }

        function closeSmartModal() {
            window.speechSynthesis.cancel(); // Stop audio immediately
            audioVisualizer.classList.add('hidden');
            
            modalContent.classList.remove('scale-100', 'opacity-100');
            modalContent.classList.add('scale-95', 'opacity-0');
            setTimeout(() => {
                smartModal.classList.add('hidden');
                resetModal();
            }, 300);
        }

        function resetModal() {
            window.speechSynthesis.cancel();
            audioVisualizer.classList.add('hidden');
            modalStep1.classList.remove('hidden');
            modalResult.classList.add('hidden');
        }

        function smartCheck(status) {
            modalStep1.classList.add('hidden');
            modalResult.classList.remove('hidden');
            
            let message = "";
            let course = "";

            if (status === 'SPM') {
                message = "Anda baru ingin memulakan kerjaya? Sebagai lepasan SPM atau setara, anda ada dua pilihan utama. Pertama, ambil Kursus Pendawai PW2 jika anda berminat dengan pendawaian rumah dan berumur 18 tahun ke atas. Kedua, ambil Kursus Penjaga Jentera A0 jika anda berumur 20 tahun ke atas dan ingin bekerja dalam penyelenggaraan sistem elektrik. Langkah ini wajib untuk permulaan kerjaya anda.";
                course = "PW2 ATAU A0";
            } else if (status === 'PW2') {
                message = "Tahniah atas kompetensi PW2 anda. Langkah seterusnya yang disyorkan adalah Kursus Pendawai Fasa Tiga, iaitu PW4. Pastikan anda mempunyai sekurang-kurangnya satu tahun pengalaman kerja menggunakan sijil PW1 atau PW2 anda sebelum memohon.";
                course = "KURSUS PW4";
            } else if (status === 'A0') {
                message = "Sebagai pemegang A0, anda mempunyai potensi besar. Anda boleh terus mengambil Kursus A1 jika anda mempunyai satu tahun pengalaman dalam Talian Atas. Atau, anda boleh mengambil jalan pantas ke A4 dengan melengkapkan Modul JKVR (Janakuasa Voltan Rendah) terlebih dahulu.";
                course = "A1 ATAU MODUL JKVR";
            } else if (status === 'A1') {
                message = "Anda sudah berada di tahap A1. Selalunya, pemegang A1 kekurangan pengalaman dalam pengendalian Janakuasa. Kami sangat mengesyorkan anda mengambil Modul JKVR untuk melengkapkan syarat pengalaman bagi memohon kompetensi A4.";
                course = "MODUL JKVR -> A4";
            } else if (status === 'A4') {
                message = "Syabas! Anda kini berada di puncak kompetensi Voltan Rendah. Cabaran seterusnya adalah dunia Voltan Tinggi. Anda layak memohon Kursus B0 11kV. Ini adalah langkah besar untuk menjadi pakar dalam industri elektrik.";
                course = "KURSUS B0 (11kV)";
            } else if (status === 'B0') {
                 message = "Anda telah mencapai tahap kompetensi Voltan Tinggi. Sila pastikan buku log anda sentiasa dikemaskini untuk pembaharuan lesen masa hadapan.";
                 course = "KOMPETEN B0";
            }

            resultText.innerText = message;
            resultCourse.innerText = course;
            
            // Trigger Voice
            speakText(message);
        }
    </script>
</body>
</html>
