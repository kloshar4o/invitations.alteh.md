<template>
    <div class="VOptions">
        <div class="constructor__menu js__scroll" :class="{active : openMenu}">
            <div class="menu">

                <div class="menu__item" data-fancybox data-src="#selectsize">
                    <span>Size</span>
                    <em>For posts {{options.size.name}} <br/> {{options.size.width}} x {{options.size.height}}px</em>
                </div>

                <div v-for="(cat, i) in [].concat(imagesData, textAreas, lists)" class="menu__item js__tab-btn"
                     v-if="cat.client || userType === 'consultant'"
                     :key="cat.id"
                     :class="{ 'active': active === i }"
                     @click="active = i">

                    <span>{{cat.menu_name}}</span>
                </div>

            </div>

        </div>

        <div :class="classes('constructor__option')">
            <div :class="classes('options js__tab-content', cat, i)"
                 v-for="(cat, i) in [].concat(imagesData, textAreas, lists)"
                 :key="cat.id"
                 :id="i">

                <div class="options__item">

                    <div class="options__title">

                        <span>{{cat.title}}</span>

                        <div class="block-range" v-show="cat.type === 'background'">
                            <label for="transparent">Opacity</label>
                            <input id="transparent" type="range" min="0" max="1" step="0.01"
                                   v-model="options.background.opacity" :disabled="!options.background.src">
                        </div>
                    </div>

                    <div :class="classes('options__content', cat, i)">
                        <div :class="classes('options__content__child', cat, i)">

                            <slot v-if="cat.type === 'textAreas'">
                                <div class="blocktextarea">
                                    <textarea :name="cat.id" :placeholder="cat.menu_name"
                                              v-model="options[cat.id]"></textarea>
                                    <span @click="options[cat.id] = ''">
                                <svg class="svg svg-x" width="50" height="50">
                                    <use xlink:href="ico/sprite/sprite.svg#x"></use>
                                </svg>
                                <em>Delete {{cat.name}}</em>
                            </span>
                                </div>
                            </slot>

                            <slot v-else-if="cat.type === 'lists'">
                                <div class="blocktext" v-for="(product, i) in options.products" :key="i">

                                    <input type="text" placeholder="Название" v-model="product.name">
                                    <input type="text" placeholder="Ссылка" v-model="product.link">

                                    <svg class="svg svg-x" width="50" height="50" @click="$delete(options.products, i)">
                                        <use xlink:href="ico/sprite/sprite.svg#x"></use>
                                    </svg>
                                </div>

                                <div class="options__add" @click="options.products.push({name:'', link:'',nameMat:'', linkMat:'',})">
                            <span>
                                <svg class="svg svg-plus" width="50" height="50">
                                    <use xlink:href="ico/sprite/sprite.svg#plus"></use>
                                </svg>
                            </span>
                                </div>
                            </slot>

                            <slot v-else>
                                <div :class="classes('options__content__image', cat, j, image.id)"
                                     v-if="image['show_'+userType]"
                                     v-for="(image, j) in orderBy(cat.images, 'sort')"
                                     :key="j"
                                     @click="setOption(cat.type, image, image.id); $emit('closeMenu')">

                                    <img :src="image.src" alt="IMG">
                                </div>
                            </slot>

                        </div>
                    </div>

                </div>
            </div>


        </div>

    </div>
</template>

<script>
    import Vue from 'vue';
    import axios from 'axios';

    import Vue2Filters from 'vue2-filters'
    Vue.use(Vue2Filters);

    export default {
        props: ['options', 'imagesData', 'textAreas', 'lists', 'openMenu', 'userType'],
        mixins: [Vue2Filters.mixin],
        data() {
            return {
                active: 0,
           }
        },
        methods: {
            classes(tagClass, cat = {}, i = 0, imageId = 0) {
                let isBg = cat.type === 'background';
                let isImg = cat.type === 'img';
                let isSvg = cat.type === 'svg';
                let classess = [tagClass];

                switch (tagClass) {
                    case 'constructor__option':
                        classess.push({
                            'active': this.openMenu
                        });
                        break;

                    case 'options js__tab-content':
                        classess.push({
                            'active': this.active === i,
                            'options-background': isBg,
                        });
                        break;

                    case 'options__content':
                        classess.push({});
                        break;

                    case 'options__content__child':
                        classess.push({
                            'scrollblock--grid': isImg || isBg,
                        });
                        break;

                    case 'options__content__image':
                        classess.push([cat.folder, {
                            'svgimg__item': isSvg,
                            'background__item': isBg,
                            'scrollblock__item': isImg,
                            'active': this.options.background.active === imageId && isBg
                        }]);
                        break;
                }
                return classess;
            },
            setOption(type, image, id) {
                let app = this;
                let maxw = 200;


                let defoultValues = {
                    x: -999,
                    y: 200,
                    angle: 0,
                };

                let img = new Image();
                img.src = image.src;

                img.onload = function () {

                    let w = this.width;
                    let h = this.height;
                    let ratio = 0;


                    if (w > maxw) {
                        ratio = maxw / w;
                        w = maxw;
                        h = h * ratio;
                    }

                    if (h < 100) {
                        h *= 2
                        w *= 2
                    }

                    defoultValues.w = w;
                    defoultValues.h = h;


                    switch (type) {
                        case 'svg':
                        case 'img':
                            app.options.drags.push({
                                ...image,
                                ...defoultValues,
                                type: type,
                                color: {},
                            });
                            break;

                        case 'background':

                            if (id === app.options.background.active) {
                                app.options.background.active = '';
                                app.options.background.src = ''
                            } else {
                                app.options.background = {
                                    opacity: app.options.background.opacity || 0.5,
                                    src: image.src,
                                    active: image.id,
                                };
                            }
                            break;

                    }
                };
            }
        },
    }
</script>
