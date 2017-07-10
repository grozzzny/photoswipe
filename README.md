```html
<!-- Root element of PhotoSwipe. Must have class pswp. -->
        <div class="pswp" tabindex="-1" role="dialog" aria-hidden="true">

            <!-- Background of PhotoSwipe.
                 It's a separate element, as animating opacity is faster than rgba(). -->
            <div class="pswp__bg"></div>

            <!-- Slides wrapper with overflow:hidden. -->
            <div class="pswp__scroll-wrap">

                <!-- Container that holds slides. PhotoSwipe keeps only 3 slides in DOM to save memory. -->
                <div class="pswp__container">
                    <!-- don't modify these 3 pswp__item elements, data is added later on -->
                    <div class="pswp__item"></div>
                    <div class="pswp__item"></div>
                    <div class="pswp__item"></div>
                </div>

                <!-- Default (PhotoSwipeUI_Default) interface on top of sliding area. Can be changed. -->
                <div class="pswp__ui pswp__ui--hidden">

                    <div class="pswp__top-bar">

                        <!--  Controls are self-explanatory. Order can be changed. -->

                        <div class="pswp__counter"></div>

                        <button class="pswp__button pswp__button--close" title="Close (Esc)"></button>

                        <button class="pswp__button pswp__button--share" title="Share"></button>

                        <button class="pswp__button pswp__button--fs" title="Toggle fullscreen"></button>

                        <button class="pswp__button pswp__button--zoom" title="Zoom in/out"></button>

                        <!-- Preloader demo https://codepen.io/dimsemenov/pen/yyBWoR -->
                        <!-- element will get class pswp__preloader--active when preloader is running -->
                        <div class="pswp__preloader">
                            <div class="pswp__preloader__icn">
                                <div class="pswp__preloader__cut">
                                    <div class="pswp__preloader__donut"></div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="pswp__share-modal pswp__share-modal--hidden pswp__single-tap">
                        <div class="pswp__share-tooltip"></div>
                    </div>

                    <button class="pswp__button pswp__button--arrow--left" title="Previous (arrow left)">
                    </button>

                    <button class="pswp__button pswp__button--arrow--right" title="Next (arrow right)">
                    </button>

                    <div class="pswp__caption">
                        <div class="pswp__caption__center"></div>
                    </div>

                </div>

            </div>

        </div>


        <div class="picture" itemscope itemtype="http://schema.org/ImageGallery">
            <figure itemprop="associatedMedia" itemscope itemtype="http://schema.org/ImageObject">
                <a href="<?=Yii::$app->params['nophoto']?>" itemprop="contentUrl" data-size="1000x667" data-index="0" data-med-size="1000x667" data-med="<?=Yii::$app->params['nophoto']?>" >
                    <img src="<?=Yii::$app->params['nophoto']?>" height="400" width="600" itemprop="thumbnail" alt="Beach">
                </a>
            </figure>
            <figure itemprop="associatedMedia" itemscope itemtype="http://schema.org/ImageObject">
                <div>
                <a href="https://pongsstroy.ru/upload/iblock/d70/d70c7c788bf81369ac487f9e91d5dafb.jpg" itemprop="contentUrl" data-size="1000x667" data-index="1" data-med-size="1000x667" data-med="<?=Yii::$app->params['nophoto']?>">
                    <div>
                        <img src="https://pongsstroy.ru/upload/iblock/d70/d70c7c788bf81369ac487f9e91d5dafb.jpg" height="400" width="600" itemprop="thumbnail" alt="">
                    </div>
                </a>
                </div>
            </figure>
        </div>


        <link href="http://photoswipe.com/dist/photoswipe.css?v=4.1.1-1.0.4" rel="stylesheet">
        <link href="http://photoswipe.com/dist/default-skin/default-skin.css?v=4.1.1-1.0.4" rel="stylesheet">
        <script src="http://light.greenwest39.ru/assets/a9482176/jquery.js"></script>
        <script src="http://photoswipe.com/dist/photoswipe.min.js?v=4.1.1-1.0.4"></script>
        <script src="http://photoswipe.com/dist/photoswipe-ui-default.min.js?v=4.1.1-1.0.4"></script>

        <script>
            (function() {

                var initPhotoSwipeFromDOM = function(gallerySelector) {

                    var parseThumbnailElements = function(el) {

                        var items = [];
                        $(gallerySelector+' a').each(function() {
                            var img = $(this).find('img');
                            items.push({
                                el: this,
                                src : $(this).attr('href'),
                                w   : $(this).data('size').split('x')[0],
                                h   : $(this).data('size').split('x')[1],
                                author: $(this).data('author'),
                                msrc: img.attr('src'),// thumbnail url
                                title: img.attr('title'),// thumbnail url
                                m: {
                                    src: $(this).data('med'),
                                    w: $(this).data('med-size').split('x')[0],
                                    h: $(this).data('med-size').split('x')[1]
                                },
                                o: {
                                    src: $(this).attr('href'),
                                    w: $(this).data('size').split('x')[0],
                                    h: $(this).data('size').split('x')[1]
                                }
                            });
                        });


                        return items;
                    };

                    // find nearest parent element
                    var closest = function closest(el, fn) {
                        return el && ( fn(el) ? el : closest(el.parentNode, fn) );
                    };

                    var photoswipeParseHash = function() {
                        var hash = window.location.hash.substring(1),
                            params = {};

                        if(hash.length < 5) { // pid=1
                            return params;
                        }

                        var vars = hash.split('&');
                        for (var i = 0; i < vars.length; i++) {
                            if(!vars[i]) {
                                continue;
                            }
                            var pair = vars[i].split('=');
                            if(pair.length < 2) {
                                continue;
                            }
                            params[pair[0]] = pair[1];
                        }

                        if(params.gid) {
                            params.gid = parseInt(params.gid, 10);
                        }

                        return params;
                    };

                    var openPhotoSwipe = function(index, galleryElement, disableAnimation, fromURL) {
                        var pswpElement = document.querySelectorAll('.pswp')[0],
                            gallery,
                            options,
                            items;

                        items = parseThumbnailElements(galleryElement);

                        // define options (if needed)
                        options = {

                            galleryUID: galleryElement.getAttribute('data-pswp-uid'),

                            getThumbBoundsFn: function(index) {
                                // See Options->getThumbBoundsFn section of docs for more info
                                var thumbnail = $(items[index].el).find('img').get(0),
                                    pageYScroll = window.pageYOffset || document.documentElement.scrollTop,
                                    rect = thumbnail.getBoundingClientRect();

                                console.log(thumbnail);

                                return {x:rect.left, y:rect.top + pageYScroll, w:rect.width};
                            },

                            addCaptionHTMLFn: function(item, captionEl, isFake) {
                                if(!item.title) {
                                    captionEl.children[0].innerText = '';
                                    return false;
                                }
                                captionEl.children[0].innerHTML = item.title +  '<br/><small>Photo: ' + item.author + '</small>';
                                return true;
                            },

                        };


                        if(fromURL) {
                            if(options.galleryPIDs) {
                                // parse real index when custom PIDs are used
                                // http://photoswipe.com/documentation/faq.html#custom-pid-in-url
                                for(var j = 0; j < items.length; j++) {
                                    if(items[j].pid == index) {
                                        options.index = j;
                                        break;
                                    }
                                }
                            } else {
                                options.index = parseInt(index, 10) - 1;
                            }
                        } else {
                            options.index = parseInt(index, 10);
                        }

                        // exit if index not found
                        if( isNaN(options.index) ) {
                            return;
                        }



                        var radios = document.getElementsByName('gallery-style');
                        for (var i = 0, length = radios.length; i < length; i++) {
                            if (radios[i].checked) {
                                if(radios[i].id == 'radio-all-controls') {

                                } else if(radios[i].id == 'radio-minimal-black') {
                                    options.mainClass = 'pswp--minimal--dark';
                                    options.barsSize = {top:0,bottom:0};
                                    options.captionEl = false;
                                    options.fullscreenEl = false;
                                    options.shareEl = false;
                                    options.bgOpacity = 0.85;
                                    options.tapToClose = true;
                                    options.tapToToggleControls = false;
                                }
                                break;
                            }
                        }

                        if(disableAnimation) {
                            options.showAnimationDuration = 0;
                        }

                        // Pass data to PhotoSwipe and initialize it
                        gallery = new PhotoSwipe( pswpElement, PhotoSwipeUI_Default, items, options);

                        // see: http://photoswipe.com/documentation/responsive-images.html
                        var realViewportWidth,
                            useLargeImages = false,
                            firstResize = true,
                            imageSrcWillChange;

                        gallery.listen('beforeResize', function() {

                            var dpiRatio = window.devicePixelRatio ? window.devicePixelRatio : 1;
                            dpiRatio = Math.min(dpiRatio, 2.5);
                            realViewportWidth = gallery.viewportSize.x * dpiRatio;


                            if(realViewportWidth >= 1200 || (!gallery.likelyTouchDevice && realViewportWidth > 800) || screen.width > 1200 ) {
                                if(!useLargeImages) {
                                    useLargeImages = true;
                                    imageSrcWillChange = true;
                                }

                            } else {
                                if(useLargeImages) {
                                    useLargeImages = false;
                                    imageSrcWillChange = true;
                                }
                            }

                            if(imageSrcWillChange && !firstResize) {
                                gallery.invalidateCurrItems();
                            }

                            if(firstResize) {
                                firstResize = false;
                            }

                            imageSrcWillChange = false;

                        });

                        gallery.listen('gettingData', function(index, item) {
                            if( useLargeImages ) {
                                item.src = item.o.src;
                                item.w = item.o.w;
                                item.h = item.o.h;
                            } else {
                                item.src = item.m.src;
                                item.w = item.m.w;
                                item.h = item.m.h;
                            }
                        });

                        gallery.init();
                    };

                    // select all gallery elements
                    $( gallerySelector + ' a').each(function (i) {
                        $(this).attr('data-pswp-uid', i);
                        $(this).on('click', function (e) {
                            openPhotoSwipe(i, this );
                            e.preventDefault();
                        });
                    });


                    // Parse URL and open gallery if it contains #&pid=3&gid=1
                    var hashData = photoswipeParseHash();
                    if(hashData.pid && hashData.gid) {
                        openPhotoSwipe( hashData.pid,  galleryElements[ hashData.gid - 1 ], true, true );
                    }
                };

                initPhotoSwipeFromDOM('.picture');

            })();

        </script>

```
