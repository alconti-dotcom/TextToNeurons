<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Text to Neurons</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
</head>
<body class="bg-[#0f141c] text-[#e1e6f0] font-sans min-h-screen p-6 flex flex-col items-center">

    <div class="w-full max-w-3xl">
        <div class="mb-8 border-b border-[#26354a] pb-6 text-center relative">
            <h1 class="text-3xl font-extrabold text-[#7aa2f7] tracking-tight">
                <i class="fa-solid fa-bolt-lightning mr-2"></i> Prompt Repository Matrix
            </h1>
            <p class="text-sm text-[#787c99] mt-2">QR Scan System • Auto-Copy Redirection Engine Active</p>
            <button id="logoutBtn" onclick="logoutAdmin()" class="hidden absolute top-0 right-0 bg-[#2e191d] border border-[#5c2630] text-[#f87171] hover:bg-[#5c2630] transition-colors text-xs font-bold px-3 py-1.5 rounded-lg">
                <i class="fa-solid fa-lock mr-1"></i> Lock Editor
            </button>
        </div>

        <div id="statusAlert" class="hidden mb-6 p-4 rounded-xl text-center font-medium border text-sm transition-all duration-300">
            <span id="statusMessage"></span>
        </div>

        <div id="adminControls" class="hidden mb-6 flex justify-end">
            <button onclick="saveAllPrompts()" class="bg-[#9ece6a] hover:bg-[#739a43] text-[#0f141c] font-bold text-sm px-5 py-2.5 rounded-xl shadow-lg flex items-center gap-2 transition-all active:scale-95">
                <i class="fa-solid fa-floppy-disk"></i> Save Updates
            </button>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-4" id="matrixContainer"></div>
    </div>

<script>
    // --- GLOBAL CONFIGURATION ---
    const ADMIN_PASSWORD = "admin123"; // CHANGE YOUR PASSWORD HERE

    // Fallback original mock values
    const defaultPrompts = {
        1: `[PLACEHOLDER PROMPT 1: Put your custom ChatGPT prompt text here`,
        2: `[PLACEHOLDER PROMPT 2: Put your custom ChatGPT prompt text here]`,
        3: `[PLACEHOLDER PROMPT 3: Put your custom ChatGPT prompt text here]`,
        4: `[PLACEHOLDER PROMPT 4: Put your custom ChatGPT prompt text here]`,
        5: `[PLACEHOLDER PROMPT 5: Put your custom ChatGPT prompt text here]`,
        6: `[PLACEHOLDER PROMPT 6: Put your custom ChatGPT prompt text here]`,
        7: `[PLACEHOLDER PROMPT 7: Put your custom ChatGPT prompt text here]`,
        8: `[PLACEHOLDER PROMPT 8: Put your custom ChatGPT prompt text here]`,
        9: `[PLACEHOLDER PROMPT 9: Put your custom ChatGPT prompt text here]`
    };

    // Pull saved updates from local storage, or fall back to defaults
    let promptDatabase = JSON.parse(localStorage.getItem('savedPrompts')) || defaultPrompts;
    let isAdminAuthenticated = false;

    const matrixContainer = document.getElementById('matrixContainer');
    const statusAlert = document.getElementById('statusAlert');
    const statusMessage = document.getElementById('statusMessage');
    const adminControls = document.getElementById('adminControls');
    const logoutBtn = document.getElementById('logoutBtn');

    // Build/Rebuild the Grid view layout matrix safely
    function buildInterfaceMatrix() {
        matrixContainer.innerHTML = '';
        Object.keys(promptDatabase).forEach(id => {
            const block = document.createElement('div');
            block.className = "bg-[#161b26] border border-[#26354a] rounded-xl p-5 hover:border-[#3b5273] transition-all relative group";
            
            // Check display text: If it's Slot 1, always reflect the defaultPrompts version on interface
            const displayText = (id == 1) ? defaultPrompts[1] : promptDatabase[id];
            
            if (isAdminAuthenticated) {
                // Editable Content Formats for Administrators
                block.innerHTML = `
                    <div class="flex items-center justify-between mb-2">
                        <span class="text-xs font-bold text-[#e0af68] uppercase tracking-wider bg-[#2e2a24] px-2 py-0.5 rounded border border-[#e0af68]/30">
                            Editing Slot #0${id}
                        </span>
                        <i class="fa-solid fa-pen text-[#e0af68]"></i>
                    </div>
                    <h3 class="text-sm font-bold text-[#e1e6f0] mb-2">Prompt ${id}</h3>
                    <textarea id="editor-${id}" class="w-full h-32 bg-[#0f141c] rounded p-2 border border-[#26354a] text-xs font-mono text-[#9ece6a] focus:outline-none focus:border-[#e0af68] resize-none">${displayText}</textarea>
                `;
            } else {
                // Standard Public Display/One-Click Formats
                block.className += " cursor-pointer";
                block.setAttribute('onclick', `manualCopy(${id})`);
                block.innerHTML = `
                    <div class="flex items-center justify-between mb-2">
                        <span class="text-xs font-bold text-[#7aa2f7] uppercase tracking-wider bg-[#1e2738] px-2 py-0.5 rounded border border-[#26354a]">
                            Slot #0${id}
                        </span>
                        <i class="fa-solid fa-copy text-[#3b5273] group-hover:text-[#7aa2f7] transition-colors"></i>
                    </div>
                    <h3 class="text-sm font-bold text-[#e1e6f0] mb-2">Prompt ${id}</h3>
                    <div class="bg-[#0f141c] rounded p-2 border border-[#1e2738]">
                        <p class="text-[11px] font-mono text-[#9ece6a] line-clamp-4 whitespace-pre-wrap">${displayText}</p>
                    </div>
                `;
            }
            matrixContainer.appendChild(block);
        });
    }

    // Checking Engine logic for incoming traffic paths
    function checkIncomingScan() {
        const urlParams = new URLSearchParams(window.location.search);
        const promptTargetId = urlParams.get('id');

        if (promptTargetId && (promptTargetId == 1 || promptDatabase[promptTargetId])) {
            // SCANNED VIA QR CODE: Create action gesture overlay block bypass
            const overlay = document.createElement('div');
            overlay.className = "fixed inset-0 bg-[#0f141c]/95 backdrop-blur-md flex flex-col items-center justify-center z-50 p-6";
            overlay.id = "gestureOverlay";
            overlay.innerHTML = `
                <div class="text-center max-w-sm">
                    <div class="w-20 h-20 bg-[#7aa2f7]/10 rounded-full flex items-center justify-center mx-auto mb-6 border border-[#7aa2f7]/30 animate-pulse">
                        <i class="fa-solid fa-arrow-pointer text-3xl text-[#7aa2f7]"></i>
                    </div>
                    <h2 class="text-xl font-bold text-white mb-2">Prompt ready to load</h2>
                    <p class="text-sm text-[#787c99] mb-6">Tap the button below to authorize your clipboard sync.</p>
                    <button onclick="executeSecureCopy('${promptTargetId}')" class="w-full bg-[#7aa2f7] hover:bg-[#565f89] text-[#0f141c] font-bold py-4 px-6 rounded-xl shadow-lg shadow-[#7aa2f7]/20 flex items-center justify-center gap-2 transition-all active:scale-95 text-base">
                        <i class="fa-solid fa-paste"></i> Copy Prompt #${promptTargetId}
                    </button>
                </div>
            `;
            document.body.appendChild(overlay);
            buildInterfaceMatrix();
        } else {
            // DIRECT TRAFFIC: Prompt security password check interface overlay layout
            spawnLoginModal();
        }
    }

    // Admin Access Modal Window Builder
    function spawnLoginModal() {
        const modal = document.createElement('div');
        modal.className = "fixed inset-0 bg-[#0f141c] flex flex-col items-center justify-center z-40 p-6";
        modal.id = "loginModal";
        modal.innerHTML = `
            <div class="w-full max-w-md bg-[#161b26] border border-[#26354a] rounded-2xl p-6 shadow-2xl text-center">
                <div class="w-14 h-14 bg-[#e0af68]/10 text-[#e0af68] rounded-full flex items-center justify-center mx-auto mb-4 border border-[#e0af68]/20">
                    <i class="fa-solid fa-shield-halved text-2xl"></i>
                </div>
                <h2 class="text-lg font-bold text-white mb-1">Access Control Gate</h2>
                <p class="text-xs text-[#787c99] mb-6">Enter system verification password to gain repository write privileges.</p>
                
                <input type="password" id="passInput" placeholder="••••••••" class="w-full text-center py-3 bg-[#0f141c] border border-[#26354a] rounded-xl text-sm mb-4 focus:outline-none focus:border-[#7aa2f7] text-white tracking-widest">
                <div id="loginError" class="hidden text-xs text-[#f87171] mb-4 font-medium"><i class="fa-solid fa-circle-exclamation mr-1"></i> Incorrect code. Access denied.</div>
                
                <div class="flex gap-2">
                    <button onclick="bypassToViewMode()" class="flex-1 bg-[#1e2738] hover:bg-[#26354a] border border-[#26354a] text-[#a9b1d6] font-semibold text-xs py-3 rounded-xl transition-all">
                        Just View Matrix
                    </button>
                    <button onclick="verifyPassword()" class="flex-1 bg-[#7aa2f7] hover:bg-[#565f89] text-[#0f141c] font-bold text-xs py-3 rounded-xl transition-all">
                        Unlock Editor
                    </button>
                </div>
            </div>
        `;
        document.body.appendChild(modal);
        
        // Enter Key listener for Password Field
        document.getElementById('passInput').addEventListener('keyup', (e) => {
            if (e.key === 'Enter') verifyPassword();
        });
    }

    // Password evaluation routine
    window.verifyPassword = function() {
        const input = document.getElementById('passInput').value;
        const err = document.getElementById('loginError');
        if (input === ADMIN_PASSWORD) {
            isAdminAuthenticated = true;
            document.getElementById('loginModal').remove();
            adminControls.classList.remove('hidden');
            logoutBtn.classList.remove('hidden');
            buildInterfaceMatrix();
        } else {
            err.classList.remove('hidden');
            document.getElementById('passInput').value = '';
        }
    };

    // Public Viewer bypass routine
    window.bypassToViewMode = function() {
        isAdminAuthenticated = false;
        document.getElementById('loginModal').remove();
        buildInterfaceMatrix();
    };

    // Write Editor modifications out into systemic persistence storage
    window.saveAllPrompts = function() {
        if (!isAdminAuthenticated) return;
        
        Object.keys(promptDatabase).forEach(id => {
            const textareaValue = document.getElementById(`editor-${id}`).value;
            promptDatabase[id] = textareaValue;
        });

        localStorage.setItem('savedPrompts', JSON.stringify(promptDatabase));
        
        // Notify Success UI
        statusAlert.className = "mb-6 p-4 rounded-xl text-center font-medium border text-sm bg-[#1b2b24] border-[#2e5c44] text-[#a3e635]";
        statusMessage.innerHTML = `<i class="fa-solid fa-floppy-disk mr-2"></i> Repository updates written to local storage matrix successfully!`;
        statusAlert.classList.remove('hidden');
        
        window.scrollTo({ top: 0, behavior: 'smooth' });
        
        setTimeout(() => {
            statusAlert.classList.add('hidden');
        }, 4000);
    };

    // Clear session storage state
    window.logoutAdmin = function() {
        isAdminAuthenticated = false;
        adminControls.classList.add('hidden');
        logoutBtn.classList.add('hidden');
        spawnLoginModal();
    };

    // Secure execution context tied directly to button click handler
    window.executeSecureCopy = function(id) {
        const text = (id == 1) ? defaultPrompts[1] : promptDatabase[id];
        navigator.clipboard.writeText(text).then(() => {
            document.getElementById('gestureOverlay').remove();
            statusAlert.className = "mb-6 p-4 rounded-xl text-center font-medium border text-sm bg-[#1b2b24] border-[#2e5c44] text-[#a3e635]";
            statusMessage.innerHTML = `<i class="fa-solid fa-circle-check mr-2"></i> Prompt #${id} successfully copied!`;
            statusAlert.classList.remove('hidden');
        });
    };

    // Card fallback manual backup option
    window.manualCopy = function(id) {
        const text = (id == 1) ? defaultPrompts[1] : promptDatabase[id];
        navigator.clipboard.writeText(text).then(() => {
            statusAlert.className = "mb-6 p-4 rounded-xl text-center font-medium border text-sm bg-[#1b2b24] border-[#2e5c44] text-[#a3e635]";
            statusMessage.innerHTML = `<i class="fa-solid fa-circle-check mr-2"></i> Prompt #${id} manual copy successful!`;
            statusAlert.classList.remove('hidden');
            window.scrollTo({ top: 0, behavior: 'smooth' });
        });
    };

    window.addEventListener('DOMContentLoaded', checkIncomingScan);
</script>
</body>
