# isaloustore.github.io
<!-- Salve como index.html e abra no navegador -->
<!doctype html>
<html lang="pt-BR">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Loja de Roupas — Demo</title>
  <style>
    :root{--accent:#111827;--muted:#6b7280}
    *{box-sizing:border-box;font-family:Inter,system-ui,Arial}
    body{margin:0;background:#f7f7f8;color:#111}
    header{display:flex;justify-content:space-between;align-items:center;padding:16px 24px;background:#fff;box-shadow:0 1px 4px rgba(0,0,0,.06)}
    .logo{font-weight:700}
    .cart-btn{position:relative;border:0;background:transparent;cursor:pointer;font-size:16px}
    .cart-count{position:absolute;right:-8px;top:-6px;background:#ef4444;color:white;border-radius:999px;padding:2px 6px;font-size:12px}
    main{max-width:1100px;margin:28px auto;padding:0 16px}
    .grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:18px}
    .card{background:#fff;border-radius:10px;padding:12px;box-shadow:0 6px 18px rgba(16,24,40,.06);display:flex;flex-direction:column;gap:10px}
    .thumb{height:220px;border-radius:8px;background-size:cover;background-position:center}
    .title{font-weight:600}
    .price{color:var(--accent);font-weight:700}
    .sizes{display:flex;gap:8px;flex-wrap:wrap}
    .size{border:1px solid #e5e7eb;padding:6px 8px;border-radius:6px;font-size:14px;cursor:pointer}
    .add{margin-top:auto;padding:10px;border:0;border-radius:8px;background:#111827;color:white;cursor:pointer}
    /* cart modal */
    .cart-modal{position:fixed;right:18px;bottom:18px;background:#fff;padding:12px;border-radius:12px;box-shadow:0 8px 30px rgba(2,6,23,.2);width:320px;max-height:70vh;overflow:auto}
    .cart-item{display:flex;gap:10px;align-items:center;border-bottom:1px solid #f1f1f1;padding:8px 0}
    .small-thumb{width:56px;height:56px;background-size:cover;border-radius:6px}
    .qty{display:flex;gap:6px;align-items:center}
    .btn-ghost{background:transparent;border:1px solid #e5e7eb;padding:6px;border-radius:6px;cursor:pointer}
  </style>
</head>
<body>
  <header>
    <div class="logo">MinhaLoja</div>
    <div>
      <button class="cart-btn" id="openCart">Carrinho 🛒 <span class="cart-count" id="cartCount">0</span></button>
    </div>
  </header>

  <main>
    <h1>Produtos</h1>
    <div class="grid" id="products"></div>
  </main>

  <!-- Cart (inicialmente escondido) -->
  <div id="cartModal" style="display:none" class="cart-modal" aria-hidden="true">
    <h3>Carrinho</h3>
    <div id="cartItems"></div>
    <div style="margin-top:12px;display:flex;justify-content:space-between;align-items:center">
      <strong>Total: R$ <span id="cartTotal">0.00</span></strong>
      <button id="checkout" class="add">Finalizar</button>
    </div>
  </div>

  <script>
    // Dados de exemplo — substitua pelos seus produtos
    const productsData = [
      { id: 'p1', name: 'Camiseta Básica Preta', price: 79.90, img: 'https://via.placeholder.com/600x600?text=Camiseta+Preta', sizes: ['P','M','G'] },
      { id: 'p2', name: 'Moletom Cinza', price: 149.90, img: 'https://via.placeholder.com/600x600?text=Moletom+Cinza', sizes: ['M','G','GG'] },
      { id: 'p3', name: 'Calça Jeans Slim', price: 199.90, img: 'https://via.placeholder.com/600x600?text=Calça+Jeans', sizes: ['38','40','42'] }
    ];

    const $ = sel => document.querySelector(sel);
    const $$ = sel => document.querySelectorAll(sel);

    const productsEl = $('#products');
    const cartCountEl = $('#cartCount');
    const cartModal = $('#cartModal');
    const cartItemsEl = $('#cartItems');
    const cartTotalEl = $('#cartTotal');

    // Carrega carrinho do localStorage
    let cart = JSON.parse(localStorage.getItem('carrinho_loja')) || [];

    function saveCart(){ localStorage.setItem('carrinho_loja', JSON.stringify(cart)); updateCartUI(); }

    function formatPrice(v){ return v.toFixed(2).replace('.', ','); }

    function renderProducts(){
      productsEl.innerHTML = '';
      for(const p of productsData){
        const div = document.createElement('div');
        div.className = 'card';
        div.innerHTML = `
          <div class="thumb" style="background-image:url('${p.img}')"></div>
          <div class="title">${p.name}</div>
          <div class="price">R$ ${formatPrice(p.price)}</div>
          <div class="sizes" data-id="${p.id}">${p.sizes.map(s => `<div class="size" data-size="${s}">${s}</div>`).join('')}</div>
          <button class="add" data-id="${p.id}">Adicionar ao carrinho</button>
        `;
        productsEl.appendChild(div);
      }
    }

    function addToCart(productId, size = null){
      const product = productsData.find(x => x.id === productId);
      if(!product) return;
      const key = `${productId}::${size||''}`;
      const existing = cart.find(i => i.key === key);
      if(existing) existing.q++;
      else cart.push({ key, id: productId, name: product.name, price: product.price, img: product.img, size, q: 1 });
      saveCart();
    }

    function updateCartUI(){
      cartCountEl.textContent = cart.reduce((s,i)=>s+i.q,0);
      cartItemsEl.innerHTML = '';
      let total = 0;
      if(cart.length === 0) cartItemsEl.innerHTML = '<p>Seu carrinho está vazio.</p>';
      for(const item of cart){
        const itemEl = document.createElement('div');
        itemEl.className = 'cart-item';
        itemEl.innerHTML = `
          <div class="small-thumb" style="background-image:url('${item.img}')"></div>
          <div style="flex:1">
            <div>${item.name} ${item.size? '('+item.size+')' : ''}</div>
            <div style="color:var(--muted)">R$ ${formatPrice(item.price)}</div>
            <div class="qty" style="margin-top:6px">
              <button class="btn-ghost" data-action="dec" data-key="${item.key}">-</button>
              <div>${item.q}</div>
              <button class="btn-ghost" data-action="inc" data-key="${item.key}">+</button>
            </div>
          </div>
        `;
        cartItemsEl.appendChild(itemEl);
        total += item.price * item.q;
      }
      cartTotalEl.textContent = formatPrice(total);
    }

    // Eventos delegados
    document.addEventListener('click', (e) => {
      // abrir fechar carrinho
      if(e.target.closest('#openCart')) {
        cartModal.style.display = cartModal.style.display === 'none' ? 'block' : 'none';
        return;
      }
      // clique em adicionar (botão)
      const addBtn = e.target.closest('.add');
      if(addBtn){
        const id = addBtn.dataset.id;
        // tenta pegar tamanho selecionado (se houver)
        const sizesEl = document.querySelector(`.sizes[data-id="${id}"]`);
        const selectedSize = sizesEl ? sizesEl.querySelector('.size.selected')?.dataset.size : null;
        addToCart(id, selectedSize);
        return;
      }
      // clicar em tamanho
      const sizeBtn = e.target.closest('.size');
      if(sizeBtn){
        // desmarca outros no mesmo container
        const parent = sizeBtn.parentElement;
        parent.querySelectorAll('.size').forEach(s => s.classList.remove('selected'));
        sizeBtn.classList.add('selected');
        return;
      }
      // alterar qtd no carrinho
      const cartAction = e.target.closest('[data-action]');
      if(cartAction){
        const key = cartAction.dataset.key;
        const action = cartAction.dataset.action;
        const idx = cart.findIndex(i => i.key === key);
        if(idx > -1){
          if(action === 'inc') cart[idx].q++;
          if(action === 'dec'){ cart[idx].q--; if(cart[idx].q <= 0) cart.splice(idx,1); }
          saveCart();
        }
        return;
      }
      // checkout (apenas exemplo)
      if(e.target.id === 'checkout'){
        alert('Aqui você chamaria a integração de pagamento / checkout.');
      }
    });

    // render inicial
    renderProducts();
    updateCartUI();
  </script>
</body>
</html>
