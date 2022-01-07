<h1>BigCommerce Stencil Cornerstone Theme</br></h1>
<p>
  <h2>card</h2>
  https://github.com/fxspec06/BigPage/blob/master/templates/components/products/card.html
  <ul>
  <li>added image hover to the data-src attribute using data-hoverimage attribute</li>
  <li>added mouseover and mouseout functions (jquery hover was doing nothing)</li>
  </ul>
</br>

```
<a href="{{url}}"
  class="card-figure__link"
  aria-label="{{> components/products/product-info}}"
  {{#if settings.data_tag_enabled}} data-event-type="product-click" {{/if}}
  >
  <div class="card-img-container">

  <!-- load the image using custom code instead of included code,
    in order to show an alt image on mouseover. 
    i tried using JQuery, and got JQuery loading fine, but hover did nothing
    also, there's a bug where a product hover image may not be defined
    if to push to live, this would need to be fixed
  -->
  <img class="card-image lazyload" 
    data-sizes="auto" 
    src="{{cdn 'img/loading.svg'}}" 
    alt="{{image.alt}}" 
    title="{{image.alt}}"
    data-src="{{getImage image}}"
    data-hoverimage="{{#replace '{:size}' images.1.data}}500x659{{/replace}}{{getImage images.1.data 'productgallery_size' (cdn theme_settings.default_image_product)}}"> 

    <script>
        // CUSTOM CODE to add image hover to the data-src attribute if necessary
        var el = document.getElementsByClassName('card-image')[0]; // class card-figure did nothing here
        var mainsrc = el.getAttribute('data-src');
        var newsrc = el.getAttribute('data-hoverimage');

        // used mouseover and mouseout because hover did nothing
        el.addEventListener('mouseover', function() {
            el.setAttribute('src', newsrc);
        });
        el.addEventListener('mouseout', function(){
            el.setAttribute('src', mainsrc);
        });
    </script>
```

<h2>bigcommerce.http</h2>
https://github.com/fxspec06/BigPage/blob/master/bigcommerce.http
this file used to create requests used to create and update the entire store
https://github.com/fxspec06/BigPage/blob/master/bigcommerce.http
<ul>
  <li>added additional protection for custom fields (may not be necessary)</li>
</ul>
</br>
<h2>category.html ** major additions</h2>
https://github.com/fxspec06/BigPage/blob/master/templates/pages/category.html
```
<!-- CUSTOM CODE TO ADD BUTTONS TO TOP OF PAGE -->
<div style="text-align:center; padding-bottom: 20px;">
    <button class="button button-icon" type="button" id="addToCart">Add All To Cart</button>
    <div id="clearCartWrapper" style="display: inline;">
        <button class="button button-icon" type="button" id="clearCart">Remove All Items</button>
    </div>
</div>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>
<script>
    $('#clearCartWrapper').hide();
    // delay 1 second to allow the page to load so we can grab the classNames.
    // not sure how to get around this the best way
    function delay() {
        let products = document.getElementsByClassName('card');
        for (let i=0; i<products.length; i++) {
            console.log(products[i].getAttribute('data-test'));
        }
        let productIDarray = [];
        for (let i = 0; i < products.length; i++) {
            productIDarray.push(products[i].getAttribute('data-test').substring(5));
        }
        
        // get the cart and cartId
        fetch('/api/storefront/cart', {
            'credentials':'include'
        }).then(function(response) {
            return response.json();
        }).then( async function(cart) {
            // only show the button if cart exists and there are items in it
            $('#clearCartWrapper').show(cart[0].lineItems);
        }).catch(function (error) {
            $('#clearCartWrapper').hide();
            console.error("failed to get cart");
            console.error(error);
        });

        // when #addToCart is clicked...
        $("button#addToCart").click(function() {
            // ADD all products in productIDarray to cart consecutively, one call at a time...
            const control = async _ => {
                for (let i = 0; i < productIDarray.length; i++) {
                    let final = false;
                    if (i == productIDarray.length - 1) final = true;

                    // call add product on cart.php async
                    await $.get("/cart.php?action=add&product_id=" + productIDarray[i])
                    .done(function(data, status, xhr) {
                        if (final == true) {
                            alert("All items added to cart.");
                            window.location = "/cart.php";
                        }
                    })
                    .fail(function(xhr, status, error) {
                        console.log('oh noes, error with status ' + status + ' and error: ');
                        console.error(error);
                        return xhr.done();
                    });
                }
            }
            control();
        });
    }
    setTimeout(delay.bind(this), 1000);

    // **** CLEAR CART BUTTON CLICK FUNCTION HERE
    $("button#clearCart").click(function() {

        // get the cart and cartId
        fetch('/api/storefront/cart', {
            'credentials':'include'
        }).then(function(response) {
            return response.json();
        }).then( async function(cart) {
            // create a 'basket' array of items
            let basket = cart[0].lineItems.physicalItems;

            if (basket.length) {
                // clear items
                let cartId = cart[0].id

                for (let i=0; i < basket.length; i++) {
                    let item = basket[i];
                    let url = '/remote/v3/carts/' + cartId + '/items/' + item.id;
                    let final = false;
                    if (i == basket.length - 1) final = true;
                    let actionUrl = "/cart.php?action=remove&item=" + item.id;

                    // call remove cart item from cart.php
                    await $.get(actionUrl)
                    .done(function(data, status, xhr) {
                        if (final == true) {
                            alert("Cart cleared.");
                            location.reload();
                        }
                    })
                    .fail(function(xhr, status, error) {
                        console.log('oh noes, error with status ' + status + ' and error: ');
                        console.error(error);
                        return xhr.done();
                    });
                }
            }
            return cartId;
        }).catch(function (error) {
            console.error("failed to get cart");
            console.error(error);
        });
    });
</script>
```

</p>
