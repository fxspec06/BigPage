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
<!-- load the image using custom code instead of included code,
    in order to show an alt image on mouseover.
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


        /*
            the following code should be updated in the future for allowing
            hovering second image on hover of products other than the first product
            i tried several ways and couldn't get it perfect in the timeframe
        var storeurl = 'https://api.bigcommerce.com/stores/e0336893-e740-4017-9afc-64ccac281365/v3/catalog/products/{{id}}/images';
            console.log("storeurl: " + storeurl);
        $.get(storeurl, { // DOES NOT WORK. got to find a way to get the catalog images request.
            'credentials':'include',
        }).then(function(response) {
            console.log(response);
            return response.json();
        }).then( async function(images) { 
            // update the new image src to set the attribute later
            newsrc = images[1].getAttribute('data-src'); // untested but should work
        }).catch(function (error) {
            console.error("failed to get images");
            console.error(error);
        });
        */

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
this file used to create requests used to create and update the entire store
https://github.com/fxspec06/BigPage/blob/master/bigcommerce.http
</br>
</br>
</br>
</br>
<h2>category.html</h2>
added two buttons to top of all category pages</br></br>
the first is add all to cart, which always displays</br></br>
the second is remove all from cart, which was a bit trickier to get working, but is dynamic in that it only displays if there are items in the cart
for both of these buttons, the more items to add or remove, the longer, because jquery ajax only allows one instance at a time, and if you're adding 12 requests that's 12 times like .7 seconds. this is the best way (maybe the only way) to accomplish this i would think
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
