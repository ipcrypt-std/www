---
layout: page
title: IPCrypt Playground
description: Try IPCrypt in your browser with this interactive playground. Encrypt and decrypt IP addresses using different modes.
permalink: /playground/
---

<div class="max-w-4xl mx-auto">
    <h1 class="text-3xl font-bold mb-6">IPCrypt Interactive Playground</h1>
    
    <p class="mb-6">
        Try IPCrypt directly in your browser! This interactive playground allows you to encrypt and decrypt IP addresses using the different encryption modes provided by IPCrypt.
    </p>
    
    <div class="card mb-8">
        <h2 class="text-2xl font-bold mb-4">Encrypt/Decrypt IP Address</h2>
        
        <div class="mb-4">
            <label for="ip-address" class="block mb-2 font-medium">IP Address:</label>
            <input type="text" id="ip-address" class="w-full p-2 border rounded" placeholder="Enter an IP address (e.g., 192.168.1.1 or 2001:db8::1)" value="192.168.1.1">
        </div>
        
        <div class="mb-4">
            <label for="encryption-key" class="block mb-2 font-medium">Encryption Key (hex):</label>
            <input type="text" id="encryption-key" class="w-full p-2 border rounded" placeholder="Enter a 16-byte hex key (32 characters)" value="000102030405060708090a0b0c0d0e0f">
            <p class="text-sm text-gray-600 mt-1">For demonstration purposes, a default key is provided. In production, use a secure random key.</p>
        </div>
        
        <div class="mb-4">
            <label for="encryption-mode" class="block mb-2 font-medium">Encryption Mode:</label>
            <select id="encryption-mode" class="w-full p-2 border rounded">
                <option value="deterministic">ipcrypt-deterministic</option>
                <option value="nd">ipcrypt-nd</option>
                <option value="ndx">ipcrypt-ndx</option>
            </select>
        </div>
        
        <div class="mb-4" id="tweak-container" style="display: none;">
            <label for="tweak" class="block mb-2 font-medium">Tweak (hex):</label>
            <input type="text" id="tweak" class="w-full p-2 border rounded" placeholder="Enter a tweak in hex format">
            <p class="text-sm text-gray-600 mt-1">Required for non-deterministic modes. For nd mode, use 8 bytes (16 hex chars). For ndx mode, use 16 bytes (32 hex chars).</p>
        </div>
        
        <div class="flex space-x-4 mb-6">
            <button id="encrypt-btn" class="btn btn-primary">Encrypt</button>
            <button id="decrypt-btn" class="btn btn-secondary">Decrypt</button>
            <button id="generate-key-btn" class="btn">Generate Random Key</button>
            <button id="generate-tweak-btn" class="btn" style="display: none;">Generate Random Tweak</button>
        </div>
        
        <div id="result-container" class="p-4 bg-gray-50 rounded border" style="display: none;">
            <h3 class="font-bold mb-2">Result:</h3>
            <p id="result-text" class="font-mono"></p>
        </div>
    </div>
    
    <div class="card mb-8">
        <h2 class="text-2xl font-bold mb-4">About This Playground</h2>
        
        <p class="mb-4">
            This playground uses the <a href="https://www.npmjs.com/package/ipcrypt" target="_blank" rel="noopener">ipcrypt</a> implementation to perform encryption and decryption operations directly in your browser.
        </p>
        
        <h3 class="text-xl font-bold mb-2">Encryption Modes</h3>
        
        <ul class="list-disc pl-6 mb-4">
            <li><strong>ipcrypt-deterministic</strong>: Format-preserving encryption that always produces the same output for the same input and key.</li>
            <li><strong>ipcrypt-nd</strong>: Non-deterministic encryption using KIASU-BC with an 8-byte tweak. Produces a 24-byte output.</li>
            <li><strong>ipcrypt-ndx</strong>: Non-deterministic encryption using AES-XTS with a 16-byte tweak. Produces a 32-byte output.</li>
        </ul>
        
        <p class="mb-4">
            For more information about the encryption modes and their properties, see the <a href="https://jedisct1.github.io/draft-denis-ipcrypt/draft-denis-ipcrypt.html" target="_blank" rel="noopener">IPCrypt specification</a>.
        </p>
    </div>
</div>

<script src="{{ site.baseurl }}/assets/js/lib/ipcrypt.js"></script>
<script>
document.addEventListener('DOMContentLoaded', function() {
    const ipAddressInput = document.getElementById('ip-address');
    const encryptionKeyInput = document.getElementById('encryption-key');
    const encryptionModeSelect = document.getElementById('encryption-mode');
    const tweakContainer = document.getElementById('tweak-container');
    const tweakInput = document.getElementById('tweak');
    const encryptBtn = document.getElementById('encrypt-btn');
    const decryptBtn = document.getElementById('decrypt-btn');
    const generateKeyBtn = document.getElementById('generate-key-btn');
    const generateTweakBtn = document.getElementById('generate-tweak-btn');
    const resultContainer = document.getElementById('result-container');
    const resultText = document.getElementById('result-text');
    
    // Show/hide tweak input based on encryption mode
    encryptionModeSelect.addEventListener('change', function() {
        if (this.value === 'deterministic') {
            tweakContainer.style.display = 'none';
            generateTweakBtn.style.display = 'none';
        } else {
            tweakContainer.style.display = 'block';
            generateTweakBtn.style.display = 'inline-flex';
            
            // Update placeholder based on mode
            if (this.value === 'nd') {
                tweakInput.placeholder = 'Enter an 8-byte tweak (16 hex characters)';
            } else if (this.value === 'ndx') {
                tweakInput.placeholder = 'Enter a 16-byte tweak (32 hex characters)';
            }
        }
    });
    
    // Generate random key
    generateKeyBtn.addEventListener('click', function() {
        const key = generateRandomHex(32); // 16 bytes = 32 hex chars
        encryptionKeyInput.value = key;
    });
    
    // Generate random tweak
    generateTweakBtn.addEventListener('click', function() {
        const mode = encryptionModeSelect.value;
        let tweakLength = 16; // 8 bytes = 16 hex chars for nd mode
        
        if (mode === 'ndx') {
            tweakLength = 32; // 16 bytes = 32 hex chars for ndx mode
        }
        
        const tweak = generateRandomHex(tweakLength);
        tweakInput.value = tweak;
    });
    
    // Encrypt button
    encryptBtn.addEventListener('click', function() {
        try {
            const ip = ipAddressInput.value.trim();
            const key = hexToBytes(encryptionKeyInput.value.trim());
            const mode = encryptionModeSelect.value;
            
            console.log('Encrypting IP:', ip);
            console.log('Key:', Array.from(key).map(b => b.toString(16).padStart(2, '0')).join(''));
            console.log('Mode:', mode);
            
            if (!ip) {
                throw new Error('Please enter an IP address');
            }
            
            if (key.length !== 16) {
                throw new Error('Key must be 16 bytes (32 hex characters)');
            }
            
            let result;
            console.log('Using ipcrypt object:', ipcrypt);
            
            if (mode === 'deterministic') {
                console.log('Using deterministic mode');
                result = ipcrypt.deterministic.encrypt(ip, key);
            } else {
                const tweak = hexToBytes(tweakInput.value.trim());
                
                if (mode === 'nd') {
                    if (tweak.length !== 8) {
                        throw new Error('Tweak must be 8 bytes (16 hex characters) for nd mode');
                    }
                    result = ipcrypt.nd.encrypt(ip, key, tweak);
                } else if (mode === 'ndx') {
                    if (tweak.length !== 16) {
                        throw new Error('Tweak must be 16 bytes (32 hex characters) for ndx mode');
                    }
                    result = ipcrypt.ndx.encrypt(ip, key, tweak);
                }
            }
            
            resultContainer.style.display = 'block';
            resultText.textContent = result;
            resultText.classList.remove('text-red-500');
        } catch (error) {
            resultContainer.style.display = 'block';
            resultText.textContent = 'Error: ' + error.message;
            resultText.classList.add('text-red-500');
        }
    });
    
    // Decrypt button
    decryptBtn.addEventListener('click', function() {
        try {
            const encryptedValue = ipAddressInput.value.trim();
            const key = hexToBytes(encryptionKeyInput.value.trim());
            const mode = encryptionModeSelect.value;
            
            if (!encryptedValue) {
                throw new Error('Please enter an encrypted value');
            }
            
            if (key.length !== 16) {
                throw new Error('Key must be 16 bytes (32 hex characters)');
            }
            
            let result;
            
            if (mode === 'deterministic') {
                result = ipcrypt.deterministic.decrypt(encryptedValue, key);
            } else {
                const tweak = hexToBytes(tweakInput.value.trim());
                
                if (mode === 'nd') {
                    if (tweak.length !== 8) {
                        throw new Error('Tweak must be 8 bytes (16 hex characters) for nd mode');
                    }
                    result = ipcrypt.nd.decrypt(encryptedValue, key, tweak);
                } else if (mode === 'ndx') {
                    if (tweak.length !== 16) {
                        throw new Error('Tweak must be 16 bytes (32 hex characters) for ndx mode');
                    }
                    result = ipcrypt.ndx.decrypt(encryptedValue, key, tweak);
                }
            }
            
            resultContainer.style.display = 'block';
            resultText.textContent = result;
            resultText.classList.remove('text-red-500');
        } catch (error) {
            resultContainer.style.display = 'block';
            resultText.textContent = 'Error: ' + error.message;
            resultText.classList.add('text-red-500');
        }
    });
    
    // Helper functions
    function generateRandomHex(length) {
        const bytes = new Uint8Array(length / 2);
        crypto.getRandomValues(bytes);
        return Array.from(bytes).map(b => b.toString(16).padStart(2, '0')).join('');
    }
    
    function hexToBytes(hex) {
        hex = hex.replace(/\s/g, ''); // Remove whitespace
        const bytes = new Uint8Array(hex.length / 2);
        for (let i = 0; i < hex.length; i += 2) {
            bytes[i / 2] = parseInt(hex.substr(i, 2), 16);
        }
        return bytes;
    }
});
</script>

<style>
.btn-sm {
    padding: 0.375rem 0.75rem;
    font-size: 0.875rem;
}

.text-red-500 {
    color: #ef4444;
}

.list-disc {
    list-style-type: disc;
}

.font-mono {
    font-family: var(--font-mono);
}
</style>