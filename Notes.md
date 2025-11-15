<!-- *Count unique items in the cart before rendering the icon and badge -->

{% assign unique_item_count = cart.items | size %}

<a
href="{{ routes.cart_url }}"
class="text-base whitespace-nowrap text-gray-500 hover:text-gray-900"

>   <span class="relative inline-block">

    {% render 'icon-shopping-bag' %}
    <span
      data-cart-count
      class="absolute -top-2 -right-2 flex h-5 w-5 items-center justify-center rounded-full text-xs font-semibold text-white {% if unique_item_count > 0 %} bg-gray-600 {% else %} bg-transparent {% endif %}"
    >
      {% if unique_item_count > 0 %}
        {{ unique_item_count }}
      {% endif %}
    </span>

  </span>
</a>
<!-- *Count unique items in the cart before rendering the icon and badge -->

<!-- *snippets/add-to-cart-popup.liquid -->
<!-- 1 -->
<div id="add-to-cart-popup" class="add-to-cart-popup" style="display: none;">
  <div class="popup-overlay"></div>
  <div class="popup-content">
    <button class="popup-close" type="button" aria-label="Close">
      &times;
    </button>
    <h3>Item added to your cart!</h3>
    <div class="popup-body">
      <p>Your item has been successfully added to the cart.</p>
      <a href="/cart" class="button button--primary">View Cart</a>
      <button class="button button--secondary popup-close">
        Continue Shopping
      </button>
    </div>
  </div>
</div>

<style>
  .add-to-cart-popup {  
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 9999;
  }
  .add-to-cart-popup .popup-overlay {
    position: absolute;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
  }
  .add-to-cart-popup .popup-content {
    position: relative;
    max-width: 400px;
    margin: 10% auto;
    background: #fff;
    padding: 2rem;
    border-radius: 8px;
    text-align: center;
    z-index: 10000;
  }
  .add-to-cart-popup .popup-close {
    background: none;
    border: none;
    font-size: 1.5rem;
    cursor: pointer;
    position: absolute;
    top: 10px;
    right: 15px;
  }
</style>

<script>
  document.addEventListener('DOMContentLoaded', function () {
    const popup = document.getElementById('add-to-cart-popup')
    const closeButtons = popup.querySelectorAll('.popup-close')

    // Close popup
    closeButtons.forEach((btn) => {
      btn.addEventListener('click', () => (popup.style.display = 'none'))
    })

    // Listen for add to cart success (Shopify AJAX API example)
    document.body.addEventListener('itemAddedToCart', () => {
      popup.style.display = 'block'
    })
  })
</script>

<script>
  const productForm = document.getElementById('product-form')

  productForm.addEventListener('submit', function (event) {
    // Optionally prevent default if using AJAX submission
    // event.preventDefault();

    // Dispatch the custom event after adding the item to cart
    document.body.dispatchEvent(new Event('itemAddedToCart'))

    // If using AJAX to update cart, you can place this inside the fetch then() block
  })
</script>

<!-- 2 - inside theme.liquid/ before </body> -->
<!-- Snippet-style section: Add-to-Cart Popup -->

{% section 'snippet-add-to-cart-popup' %}

<!-- *snippets/add-to-cart-popup.liquid -->

<!-- *Fly to cart instruction -->
<!-- 1 - inside assets/ fly-to-cart.css -->

.fly-image {
pointer-events: none;
border-radius: 8px;
position: fixed;
transition: all 0.8s ease-in-out;
z-index: 9999;
}

#cart-popup {
position: fixed;
top: 20px;
right: 20px;
background: black;
color: white;
padding: 10px 20px;
border-radius: 6px;
opacity: 0;
transform: translateY(-20px);
transition: all 0.3s ease-in-out;
z-index: 9999;
}

#cart-popup.show {
opacity: 1;
transform: translateY(0);
}

<!-- 2 - inside snippets/ fly-to-cart.liquid -->
<link rel="stylesheet" href="{{ 'fly-to-cart.css' | asset_url }}" media="all">

<div id="cart-popup" class="cart-popup">
  <span>Item added to your cart!</span>
</div>

<script>
  document.addEventListener('DOMContentLoaded', () => {
    const popup = document.getElementById('cart-popup')

    document.body.addEventListener('click', (event) => {
      const addToCartBtn = event.target.closest(
        'form[action="/cart/add"] [type="submit"], .product-form__submit, button[name="add"]',
      )
      if (!addToCartBtn) return

      const productForm = addToCartBtn.closest('form')
      if (!productForm) return

      let productImage

      if (window.innerWidth >= 768) {
        productImage = Array.from(productForm.querySelectorAll('img')).find(
          (img) => img.offsetParent !== null,
        )
      } else {
        const activeSlide = document.querySelector('.swiper-slide-active img')
        if (activeSlide) productImage = activeSlide
      }

      // fallback: first visible product image anywhere
      if (!productImage) {
        productImage = Array.from(
          document.querySelectorAll('.product--medias img'),
        ).find((img) => img.offsetParent !== null)
      }

      if (!productImage) return

      // Find the visible cart button
      const cartLink = Array.from(
        document.querySelectorAll('a[href="{{ routes.cart_url }}"]'),
      ).find((el) => el.offsetParent !== null)
      if (!cartLink) return

      const imgRect = productImage.getBoundingClientRect()
      const cartRect = cartLink.getBoundingClientRect()

      const flyingImg = productImage.cloneNode(true)
      flyingImg.classList.add('fly-image')
      flyingImg.style.left = `${imgRect.left}px`
      flyingImg.style.top = `${imgRect.top}px`
      flyingImg.style.width = `${imgRect.width}px`
      flyingImg.style.height = `${imgRect.height}px`
      document.body.appendChild(flyingImg)

      requestAnimationFrame(() => {
        flyingImg.style.left = `${cartRect.left + cartRect.width / 2 - 20}px`
        flyingImg.style.top = `${cartRect.top + cartRect.height / 2 - 20}px`
        flyingImg.style.width = '40px'
        flyingImg.style.height = '40px'
        flyingImg.style.opacity = '0.3'
        flyingImg.style.transform = 'rotate(20deg)'
      })

      flyingImg.addEventListener('transitionend', () => {
        flyingImg.remove()
        if (popup) {
          popup.classList.add('show')
          setTimeout(() => popup.classList.remove('show'), 2000)
        }
      })
    })
  })
</script>

<!-- 3 at the bottom of the template, just before </div> add: -->

{% render 'fly-to-cart' %} -->

<!-- *Fly to cart instruction -->
