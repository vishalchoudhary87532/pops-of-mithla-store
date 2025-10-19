# pops-of-mithla-store
Pops of Mithila: The global e-commerce platform for GI-Tagged Makhana (Fox Nuts) sourced directly from the sacred ponds of Mithila. We blend ancient superfood purity with bold global flavors. This repository contains the full single-file web app with live stock, admin, and order management features.
<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pops of Mithila - Global Makhana Store</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Inter Font -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --color-primary: #8C477D; /* Deep Lotus Pink/Purple */
            --color-secondary: #FFD700; /* Gold/Saffron */
            --color-bg: #F8F4F9; /* Off-White/Cream */
        }
        body {
            font-family: 'Inter', sans-serif;
            background-color: var(--color-bg);
            color: #333;
        }
        .btn-primary {
            background-color: var(--color-primary);
            color: white;
            transition: background-color 0.3s;
        }
        .btn-primary:hover {
            background-color: #6a345e;
        }
        .badge-gi {
            background-color: var(--color-secondary);
            color: #333;
        }
    </style>
    <!-- Firebase Imports -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, onSnapshot, collection, query, addDoc, serverTimestamp, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // --- Global Firebase & App Variables (MANDATORY USE) ---
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        
        let db;
        let auth;
        let userId = null;
        let cartCollectionRef;
        let ordersCollectionRef;
        let isAuthReady = false;

        // Mock Product Data
        const products = [
            { id: 'p01', name_hi: 'सादा पॉप्‍स', name_en: 'Plain Pops', flavor_hi: 'क्लासिक, बिना फ्लेवर', flavor_en: 'Classic, Unflavored', price: 350, image: 'https://placehold.co/150x150/f0f0f0/333?text=Plain' },
            { id: 'p02', name_hi: 'चटपटा चाट', name_en: 'Tangy Chaat', flavor_hi: 'तीखा भारतीय मसाला', flavor_en: 'Spicy Indian Masala', price: 390, image: 'https://placehold.co/150x150/f0f0f0/333?text=Chaat' },
            { id: 'p03', name_hi: 'देसी घी रोस्ट', name_en: 'Desi Ghee Roast', flavor_hi: 'शुद्ध घी में भुना हुआ', flavor_en: 'Roasted in Pure Ghee', price: 450, image: 'https://placehold.co/150x150/f0f0f0/333?text=Ghee' },
            { id: 'p04', name_hi: 'रसीला कैरेमल', name_en: 'Caramel Delight', flavor_hi: 'मीठा और कुरकुरा', flavor_en: 'Sweet & Crunchy', price: 420, image: 'https://placehold.co/150x150/f0f0f0/333?text=Caramel' },
        ];

        let cartItems = {};
        let currentLang = 'hi'; // Default language is Hindi

        // UI Text strings for translation
        const langStrings = {
            'hi': {
                appName: 'पॉप्स ऑफ मिथिला',
                tagline: 'पवित्र जल का सुपरफूड, वैश्विक स्वाद के साथ',
                giBadge: 'GI-टैग प्रमाणिकता',
                cartTitle: 'आपका कार्ट',
                cartEmpty: 'कार्ट खाली है। खरीदारी शुरू करें!',
                total: 'कुल राशि',
                checkout: 'आगे बढ़ें और ऑर्डर प्लेस करें',
                productsTitle: 'हमारे पॉप्स',
                addToCart: 'कार्ट में जोड़ें',
                removeFromCart: 'हटाएँ',
                quantity: 'मात्रा',
                storyTitle: 'मिथिला की कहानी',
                storyContent: 'मखाना मिथिला की सदियों पुरानी पहचान है। "पग-पग पोखरि, माछ मखान" (हर कदम पर ताल, मछली और मखाना) यह यहां की संस्कृति का प्रतीक है। हमारा मखाना सीधे पवित्र झीलों से आता है और कारीगरों के हाथों से तैयार किया जाता है।',
                userIdDisplay: 'यूज़र ID (पहचान)',
                orderFormTitle: 'ऑर्डर प्लेसमेंट',
                name: 'आपका नाम',
                address: 'शिपिंग पता (देश सहित)',
                placeOrder: 'ऑर्डर फाइनल करें',
                orderSuccess: 'आपका ऑर्डर सफलतापूर्वक रिकॉर्ड किया गया है। हम जल्द ही आपसे संपर्क करेंगे!',
                orderFailed: 'ऑर्डर प्लेस करने में विफल रहा।',
                error: 'त्रुटि',
                close: 'बंद करें',
            },
            'en': {
                appName: 'Pops of Mithila',
                tagline: 'The Superfood of Sacred Waters, with Global Flavors',
                giBadge: 'GI-Tagged Authenticity',
                cartTitle: 'Your Cart',
                cartEmpty: 'Cart is empty. Start shopping!',
                total: 'Total',
                checkout: 'Proceed to Checkout',
                productsTitle: 'Our Pops',
                addToCart: 'Add to Cart',
                removeFromCart: 'Remove',
                quantity: 'Quantity',
                storyTitle: 'The Story of Mithila',
                storyContent: 'Makhana is the age-old identity of Mithila. "Pag-pag pokhari, machh makhan" (Ponds, fish, and makhana at every step) is the symbol of its culture. Our makhana comes directly from the sacred ponds and is prepared by the hands of skilled artisans.',
                userIdDisplay: 'User ID (Identifier)',
                orderFormTitle: 'Order Placement',
                name: 'Your Name',
                address: 'Shipping Address (Include Country)',
                placeOrder: 'Finalize Order',
                orderSuccess: 'Your order has been successfully recorded. We will contact you shortly!',
                orderFailed: 'Failed to place order.',
                error: 'Error',
                close: 'Close',
            }
        };

        // --- Utility Functions ---

        /** Fetches string based on current language */
        const T = (key) => langStrings[currentLang][key] || key;

        /** Updates all dynamic UI text based on currentLang */
        function updateUIStrings() {
            document.querySelectorAll('[data-lang-key]').forEach(el => {
                const key = el.getAttribute('data-lang-key');
                el.textContent = T(key);
            });
            // Update cart/product UI after language change
            renderProducts();
            renderCart();
            // Update HTML lang attribute
            document.documentElement.lang = currentLang;
        }

        function showMessage(title, message) {
            const modal = document.getElementById('message-modal');
            document.getElementById('message-modal-title').textContent = title;
            document.getElementById('message-modal-body').textContent = message;
            modal.classList.remove('hidden');
        }

        function hideMessage() {
            document.getElementById('message-modal').classList.add('hidden');
        }

        // --- Firebase/Auth Setup ---

        function initFirebase() {
            if (!firebaseConfig) {
                console.error("Firebase config not available.");
                showMessage(T('error'), 'Firebase configuration is missing. Cannot proceed with data operations.');
                return;
            }

            try {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                setLogLevel('Debug');
                
                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        console.log("Authenticated with UID:", userId);
                        cartCollectionRef = collection(db, `artifacts/${appId}/users/${userId}/makhana_cart`);
                        ordersCollectionRef = collection(db, `artifacts/${appId}/users/${userId}/makhana_orders`);
                        document.getElementById('user-id-display').textContent = `${T('userIdDisplay')}: ${userId}`;
                        isAuthReady = true;
                        
                        // Start listening to the cart immediately after auth
                        setupCartListener();
                    } else {
                        // Sign in anonymously if no custom token is available
                        if (initialAuthToken) {
                            await signInWithCustomToken(auth, initialAuthToken);
                        } else {
                            await signInAnonymously(auth);
                        }
                    }
                });
            } catch (error) {
                console.error("Firebase initialization failed:", error);
                showMessage(T('error'), `Firebase initialization error: ${error.message}`);
            }
        }

        // --- Cart Logic (Firestore) ---

        function setupCartListener() {
            if (!isAuthReady) return;
            const q = query(cartCollectionRef);
            
            onSnapshot(q, (snapshot) => {
                cartItems = {};
                snapshot.forEach(doc => {
                    const item = doc.data();
                    if (item.quantity > 0) {
                        cartItems[doc.id] = { ...item, docId: doc.id };
                    }
                });
                renderCart();
            }, (error) => {
                console.error("Error setting up cart listener:", error);
                showMessage(T('error'), 'Could not load cart data in real-time.');
            });
        }

        async function addToCart(productId) {
            if (!isAuthReady) {
                showMessage(T('error'), 'Authentication not ready. Please wait.');
                return;
            }
            try {
                const product = products.find(p => p.id === productId);
                if (!product) return;

                const docRef = doc(cartCollectionRef, productId);
                const currentItem = cartItems[productId];
                
                const newQuantity = (currentItem ? currentItem.quantity : 0) + 1;

                await setDoc(docRef, {
                    productId: product.id,
                    name_hi: product.name_hi,
                    name_en: product.name_en,
                    price: product.price,
                    quantity: newQuantity,
                });

            } catch (error) {
                console.error("Error adding to cart:", error);
                showMessage(T('error'), 'Item could not be added to the cart.');
            }
        }

        async function updateCartQuantity(productId, newQuantity) {
            if (!isAuthReady) return;
            try {
                const docRef = doc(cartCollectionRef, productId);

                if (newQuantity <= 0) {
                    // Remove item if quantity is zero or less
                    await setDoc(docRef, { quantity: 0 }, { merge: true }); // Using setDoc to set quantity to 0 for listener
                } else {
                    await setDoc(docRef, { quantity: newQuantity }, { merge: true });
                }

            } catch (error) {
                console.error("Error updating cart quantity:", error);
                showMessage(T('error'), 'Could not update item quantity.');
            }
        }

        // --- Order Placement ---

        function showOrderModal() {
            if (Object.keys(cartItems).length === 0) {
                showMessage(T('error'), 'कृपया पहले कार्ट में आइटम जोड़ें!');
                return;
            }
            document.getElementById('order-modal').classList.remove('hidden');
            document.getElementById('cart-modal').classList.add('hidden');
        }

        function hideOrderModal() {
            document.getElementById('order-modal').classList.add('hidden');
        }

        async function placeOrder() {
            if (!isAuthReady) {
                showMessage(T('error'), 'Authentication not ready.');
                return;
            }

            const name = document.getElementById('customer-name').value;
            const address = document.getElementById('shipping-address').value;

            if (!name || !address) {
                showMessage(T('error'), 'कृपया अपना नाम और शिपिंग पता दर्ज करें।');
                return;
            }

            const total = Object.values(cartItems).reduce((sum, item) => sum + item.price * item.quantity, 0);

            try {
                await addDoc(ordersCollectionRef, {
                    userId: userId,
                    customerName: name,
                    shippingAddress: address,
                    items: Object.values(cartItems).map(item => ({
                        productId: item.productId,
                        name: T('appName') === 'Pops of Mithila' ? item.name_en : item.name_hi, // Save the name based on the current app language for easy viewing
                        price: item.price,
                        quantity: item.quantity
                    })),
                    totalAmount: total,
                    orderDate: serverTimestamp(),
                    status: 'Pending',
                });
                
                // Clear the cart by setting all quantities to 0
                const cartUpdates = Object.keys(cartItems).map(productId => {
                    const docRef = doc(cartCollectionRef, productId);
                    return setDoc(docRef, { quantity: 0 }, { merge: true });
                });
                await Promise.all(cartUpdates);


                hideOrderModal();
                showMessage(T('orderSuccess'), `${T('total')}: ₹${total}`);
                
            } catch (error) {
                console.error("Error placing order:", error);
                showMessage(T('orderFailed'), `Error: ${error.message}`);
            }
        }

        // --- Render UI ---

        function renderProducts() {
            const productGrid = document.getElementById('product-grid');
            productGrid.innerHTML = ''; // Clear existing content

            products.forEach(product => {
                const name = currentLang === 'hi' ? product.name_hi : product.name_en;
                const flavor = currentLang === 'hi' ? product.flavor_hi : product.flavor_en;
                const cartQty = cartItems[product.id] ? cartItems[product.id].quantity : 0;
                
                const card = `
                    <div class="bg-white p-4 rounded-xl shadow-lg flex flex-col items-center border border-gray-100 transition duration-300 hover:shadow-xl">
                        <img src="${product.image}" alt="${name}" class="w-24 h-24 rounded-full mb-3 border-4 border-gray-100">
                        <h3 class="text-xl font-semibold mb-1 text-gray-800 text-center">${name}</h3>
                        <p class="text-sm text-gray-500 mb-2 text-center">${flavor}</p>
                        <p class="text-2xl font-bold text-red-600 mb-4">₹${product.price}</p>
                        
                        ${cartQty > 0 
                            ? `<div class="flex items-center space-x-2 text-green-600 font-semibold mb-2">
                                <span data-lang-key="quantity">${T('quantity')}</span>: ${cartQty}
                            </div>
                            <div class="flex space-x-2 w-full justify-center">
                                <button onclick="updateCartQuantity('${product.id}', ${cartQty - 1})" class="bg-red-500 hover:bg-red-600 text-white font-bold py-1 px-3 rounded-full text-lg">-</button>
                                <button onclick="updateCartQuantity('${product.id}', ${cartQty + 1})" class="bg-green-500 hover:bg-green-600 text-white font-bold py-1 px-3 rounded-full text-lg">+</button>
                            </div>`
                            : `<button onclick="addToCart('${product.id}')" class="btn-primary font-bold py-2 px-6 rounded-full shadow-md w-full" data-lang-key="addToCart">${T('addToCart')}</button>`
                        }
                    </div>
                `;
                productGrid.insertAdjacentHTML('beforeend', card);
            });
        }

        function renderCart() {
            const cartList = document.getElementById('cart-items-list');
            const cartTotalEl = document.getElementById('cart-total');
            const cartCountEl = document.getElementById('cart-count');
            
            let total = 0;
            let totalCount = 0;

            cartList.innerHTML = '';
            
            const cartItemsArray = Object.values(cartItems);

            if (cartItemsArray.length === 0) {
                cartList.innerHTML = `<p class="text-center py-4 text-gray-500" data-lang-key="cartEmpty">${T('cartEmpty')}</p>`;
                document.getElementById('checkout-button').disabled = true;
            } else {
                document.getElementById('checkout-button').disabled = false;
                cartItemsArray.forEach(item => {
                    const name = currentLang === 'hi' ? item.name_hi : item.name_en;
                    const subtotal = item.price * item.quantity;
                    total += subtotal;
                    totalCount += item.quantity;

                    const listItem = `
                        <li class="flex justify-between items-center py-3 border-b border-gray-200">
                            <div>
                                <p class="font-semibold text-gray-800">${name}</p>
                                <p class="text-sm text-gray-500">₹${item.price} x ${item.quantity}</p>
                            </div>
                            <div class="flex items-center space-x-3">
                                <p class="font-bold">₹${subtotal}</p>
                                <button onclick="updateCartQuantity('${item.productId}', 0)" class="text-red-500 hover:text-red-700 text-sm font-semibold" data-lang-key="removeFromCart">${T('removeFromCart')}</button>
                            </div>
                        </li>
                    `;
                    cartList.insertAdjacentHTML('beforeend', listItem);
                });
            }

            cartTotalEl.textContent = `₹${total}`;
            cartCountEl.textContent = totalCount;

            // Re-render product grid to update quantities
            renderProducts();
        }

        function toggleLang() {
            currentLang = currentLang === 'hi' ? 'en' : 'hi';
            updateUIStrings();
            document.getElementById('lang-toggle').textContent = currentLang === 'hi' ? 'English' : 'हिंदी';
        }

        function toggleCartModal() {
            document.getElementById('cart-modal').classList.toggle('hidden');
        }

        // --- Initialize on Load ---
        window.onload = function() {
            initFirebase();
            updateUIStrings();
            document.getElementById('lang-toggle').textContent = 'English'; // Initial state shows the toggle button to switch to English

            // Attach event listeners for modals
            document.getElementById('cart-button').addEventListener('click', toggleCartModal);
            document.getElementById('cart-close-button').addEventListener('click', toggleCartModal);
            document.getElementById('checkout-button').addEventListener('click', showOrderModal);
            document.getElementById('order-close-button').addEventListener('click', hideOrderModal);
            document.getElementById('lang-toggle').addEventListener('click', toggleLang);
            document.getElementById('final-order-button').addEventListener('click', placeOrder);
            document.getElementById('message-close-button').addEventListener('click', hideMessage);
            
            // Expose functions globally for inline HTML events
            window.addToCart = addToCart;
            window.updateCartQuantity = updateCartQuantity;
        };

    </script>
</head>
<body>

    <!-- Header / Navbar -->
    <header class="sticky top-0 z-10 bg-white shadow-md">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-4">
                <h1 class="text-3xl font-extrabold" style="color: var(--color-primary);" data-lang-key="appName">पॉप्स ऑफ मिथिला</h1>
                <span class="badge-gi text-xs font-bold py-1 px-3 rounded-full hidden sm:inline-block" data-lang-key="giBadge">GI-टैग प्रमाणिकता</span>
            </div>

            <div class="flex items-center space-x-3">
                <button id="lang-toggle" class="bg-gray-200 text-gray-800 text-sm font-medium py-2 px-4 rounded-lg hover:bg-gray-300 transition">English</button>
                
                <button id="cart-button" class="relative p-2 rounded-full hover:bg-gray-100 transition">
                    <!-- Icon for Cart (simple SVG for reliability) -->
                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6 text-gray-600">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M2.25 3h1.386c.51 0 .955.343 1.023.835l.236 1.182c.38.19.782.35 1.196.486L7.5 8.25h1.258c.956 0 1.83.473 2.375 1.25.545.777.674 1.706.35 2.53l-1.636 4.091a2.25 2.25 0 0 0 2.146 3.033h12.115a2.25 2.25 0 0 0 2.146-3.033l-1.636-4.091a2.25 2.25 0 0 0-2.375-1.25H7.5l-.236-1.182a1.5 1.5 0 0 1-.486-.486L5.386 4.135A.75.75 0 0 0 4.5 3H2.25" />
                    </svg>
                    <span id="cart-count" class="absolute top-0 right-0 inline-flex items-center justify-center px-2 py-1 text-xs font-bold leading-none text-white transform translate-x-1/2 -translate-y-1/2 bg-red-600 rounded-full">0</span>
                </button>
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6">

        <!-- Banner / Tagline -->
        <div class="bg-gradient-to-r from-pink-100 to-white p-6 sm:p-10 rounded-xl shadow-lg mb-8 text-center border-t-4" style="border-color: var(--color-primary);">
            <p class="text-lg sm:text-2xl font-semibold mb-2" style="color: var(--color-primary);" data-lang-key="tagline">पवित्र जल का सुपरफूड, वैश्विक स्वाद के साथ</p>
            <p id="user-id-display" class="text-xs text-gray-500 mt-2"></p>
        </div>

        <!-- Mithila Story Section -->
        <section class="mb-10 bg-white p-6 rounded-xl shadow-md border-l-4" style="border-color: var(--color-secondary);">
            <h2 class="text-2xl font-bold mb-3" style="color: var(--color-primary);" data-lang-key="storyTitle">मिथिला की कहानी</h2>
            <p class="text-gray-600 text-base leading-relaxed" data-lang-key="storyContent">मखाना मिथिला की सदियों पुरानी पहचान है। "पग-पग पोखरि, माछ मखान" (हर कदम पर ताल, मछली और मखाना) यह यहां की संस्कृति का प्रतीक है। हमारा मखाना सीधे पवित्र झीलों से आता है और कारीगरों के हाथों से तैयार किया जाता है।</p>
        </section>

        <!-- Product Grid -->
        <h2 class="text-3xl font-bold mb-6 text-center" data-lang-key="productsTitle">हमारे पॉप्स</h2>
        <section id="product-grid" class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
            <!-- Products will be rendered here by JavaScript -->
        </section>

    </main>

    <!-- Cart Modal -->
    <div id="cart-modal" class="fixed inset-0 bg-gray-900 bg-opacity-75 hidden z-20">
        <div class="fixed right-0 top-0 h-full w-full sm:w-96 bg-white shadow-2xl overflow-y-auto transform transition-transform duration-300">
            <div class="p-6">
                <div class="flex justify-between items-center border-b pb-4 mb-4">
                    <h3 class="text-2xl font-bold" data-lang-key="cartTitle">आपका कार्ट</h3>
                    <button id="cart-close-button" class="text-gray-400 hover:text-gray-800">
                        <!-- Close Icon -->
                        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
                        </svg>
                    </button>
                </div>
                
                <ul id="cart-items-list" class="divide-y divide-gray-100">
                    <!-- Cart items rendered here -->
                </ul>

                <div class="mt-6 pt-4 border-t-2 border-dashed border-gray-300">
                    <div class="flex justify-between items-center text-xl font-bold mb-4">
                        <span data-lang-key="total">कुल राशि</span>:
                        <span id="cart-total" style="color: var(--color-primary);">₹0</span>
                    </div>
                    <button id="checkout-button" class="btn-primary font-bold py-3 rounded-xl w-full text-lg shadow-lg disabled:opacity-50" data-lang-key="checkout">आगे बढ़ें और ऑर्डर प्लेस करें</button>
                </div>
            </div>
        </div>
    </div>

    <!-- Order Placement Modal -->
    <div id="order-modal" class="fixed inset-0 bg-gray-900 bg-opacity-75 hidden z-30 flex items-center justify-center">
        <div class="bg-white p-6 rounded-xl shadow-2xl w-full max-w-md m-4">
            <div class="flex justify-between items-center border-b pb-3 mb-4">
                <h3 class="text-xl font-bold" data-lang-key="orderFormTitle">ऑर्डर प्लेसमेंट</h3>
                <button id="order-close-button" class="text-gray-400 hover:text-gray-800">
                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
                    </svg>
                </button>
            </div>
            
            <form id="order-form" onsubmit="event.preventDefault();">
                <div class="mb-4">
                    <label for="customer-name" class="block text-sm font-medium text-gray-700" data-lang-key="name">आपका नाम</label>
                    <input type="text" id="customer-name" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-3 focus:ring-primary focus:border-primary" required>
                </div>
                <div class="mb-6">
                    <label for="shipping-address" class="block text-sm font-medium text-gray-700" data-lang-key="address">शिपिंग पता (देश सहित)</label>
                    <textarea id="shipping-address" rows="3" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm p-3 focus:ring-primary focus:border-primary" required></textarea>
                </div>
                <button type="button" id="final-order-button" class="btn-primary font-bold py-3 rounded-xl w-full text-lg shadow-lg" data-lang-key="placeOrder">ऑर्डर फाइनल करें</button>
            </form>
        </div>
    </div>
    
    <!-- Message Modal (Replaces alert()) -->
    <div id="message-modal" class="fixed inset-0 bg-gray-900 bg-opacity-75 hidden z-40 flex items-center justify-center">
        <div class="bg-white p-6 rounded-xl shadow-2xl w-full max-w-xs m-4 text-center">
            <h3 id="message-modal-title" class="text-xl font-bold mb-3" style="color: var(--color-primary);">संदेश</h3>
            <p id="message-modal-body" class="text-gray-700 mb-6"></p>
            <button id="message-close-button" class="btn-primary font-bold py-2 px-6 rounded-full" data-lang-key="close">बंद करें</button>
        </div>
    </div>
</body>
</html>
