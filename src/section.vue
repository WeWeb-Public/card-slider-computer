<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div class="card-slider-computer">
        <!-- wwManager:start -->
        <wwSectionEditMenu :sectionCtrl="sectionCtrl" :options="openOptions"></wwSectionEditMenu>
        <!-- wwManager:end -->

        <div class="section-container">
            <wwObject class="background" :ww-object="section.data.bg" ww-category="background"></wwObject>
            <!--TOP WWOBJS-->
            <div class="top-ww-objs">
                <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="section.data.topWwObjs" class="top-ww-obj" @ww-add="add(section.data.topWwObjs, $event)" @ww-remove="remove(section.data.topWwObjs, $event)">
                    <wwObject v-for="topWwObj in section.data.topWwObjs" :key="topWwObj.uniqueId" :ww-object="topWwObj"></wwObject>
                </wwLayoutColumn>
            </div>
            <div class="full-container">
                <div class="fade right" v-show="backgroundColor" @click="nextSlide()" :style="{'background': 'linear-gradient(to right, transparent 0%,' + backgroundColor + ' 100%)'}"></div>
                <div class="fade left" v-show="backgroundColor" @click="prevSlide()" :style="{'background': 'linear-gradient(to right,' + backgroundColor + ' 0%, transparent 100%)'}"></div>

                <v-touch ref="swiper" :enabled="!editMode" @swipeleft="nextSlide()" @swiperight="prevSlide()" :swipe-options="{ direction: 'horizontal', threshold: 10, velocity: 0.2 }" class="container">
                    <div class="container-center" :style="[mobileStyle, mobileTransition]">
                        <div class="thumbnail-container" v-for="(feature, index) in section.data.features" :key="feature.uniqueId" :style="cardWidth">
                            <div class="thumbnail">
                                <!-- wwManager:start -->
                                <wwContextMenu tag="div" class="contextmenu contextmenu-center" v-if="editMode" :options="featureOptions" @ww-add-before="addFeature(index, 'before')" @ww-add-after="addFeature(index, 'after')" @ww-remove="removeFeature(index)" @ww-select-image="setFeatureImage(index)">
                                    <div class="wwi wwi-config"></div>
                                </wwContextMenu>
                                <!-- wwManager:end -->
                                <wwObject class="background" :ww-object="feature.background" ww-category="background"></wwObject>
                                <!-- feature -->
                                <div class="computer desktop twic" data-background="url(https://cdn.weweb.app/public/images/macbook.png)">
                                    <div class="website-preview" :class="{scrolling: sliderPosition == index}">
                                        <div class="preview twic" :data-background="'url(' + feature.preview + ')'"></div>
                                    </div>
                                </div>
                                <div class="computer mobile">
                                    <div class="website-preview" :class="{scrolling: sliderPosition == index}">
                                        <div class="preview twic" :data-background="'url(' + feature.preview + ')'"></div>
                                    </div>
                                </div>
                                <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="feature.contents" class="content" @ww-add="add(feature.contents, $event)" @ww-remove="remove(feature.contents, $event)">
                                    <wwObject tag="div" v-for="content in feature.contents" :key="content.uniqueId" :ww-object="content"></wwObject>
                                </wwLayoutColumn>
                            </div>
                        </div>
                    </div>
                    <div class="content-dots-wrapper">
                        <li v-for="(dot, index) in section.data.features" class="content-dot" :style="{'background': ((sliderPosition == index) ? section.data.dotColor : ''), 'border-color': section.data.dotColor}" :key="dot.uniqueId">
                            <div class="dot" @click="switchToIndex(sliderPosition, index)"></div>
                        </li>
                    </div>
                </v-touch>
            </div>

            <!--BOTTOM WWOBJS-->
            <div class="bottom-ww-objs">
                <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="section.data.bottomWwObjs" class="top-ww-obj" @ww-add="add(section.data.bottomWwObjs, $event)" @ww-remove="remove(section.data.bottomWwObjs, $event)">
                    <wwObject v-for="bottomWwObj in section.data.bottomWwObjs" :key="bottomWwObj.uniqueId" :ww-object="bottomWwObj"></wwObject>
                </wwLayoutColumn>
            </div>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>

const VueTouch = require('vue-touch')

Vue.use(VueTouch, { name: 'v-touch' })

/* wwManager:start */
import { getNewFeature } from "./defaultFeature"
/* wwManager:end */

export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            sliderPosition: 0,
            cardPercent: 100,

            /* wwManager:start */

            featureOptions: {
                name: {  //Nom du popup, si vide le popup s'appelle 'Menu'
                    en: 'Card',
                    fr: 'Carte'
                },
                items: [  //Liste des options dans le popup
                    {
                        text: {
                            en: 'Image',
                            fr: 'Image'
                        },
                        icon: 'wwi wwi-image',
                        action: 'ww-select-image'
                    }
                ]
            },
            /* wwManager:end */
        }
    },
    computed: {
        //Get the section object here !
        // It contains all the info and data about the section
        // Use it has you like !
        section() {
            return this.sectionCtrl.get();
        },
        editMode() {
            return this.sectionCtrl.getEditMode() == 'CONTENT'
        },
        featuresLength() {
            return this.section.data.features.length
        },
        mobileStyle() {

            return { 'width': (this.featuresLength * this.cardPercent) + '%' }
        },
        mobileTransition() {
            return { 'transform': 'translate(' + (- this.sliderPosition * (100 / this.featuresLength)) + '%, 0)' }
        },
        cardWidth() {
            return { 'width': (100 / this.featuresLength) + '%' }
        },
        backgroundColor() {
            const bg = wwLib.$store.getters["websiteData/getWwObject"](this.section.data.bg.uniqueId);
            if (bg && bg.content && bg.content.type == 'ww-color') {
                return bg.content.data.backgroundColor == 'transparent' ? 'white' : bg.content.data.backgroundColor;
            }
            return null;
        }
    },

    created() {


        let needUpdate = false
        //Initialize section data
        this.section.data = this.section.data || {};
        this.section.data.dotColor = this.section.data.dotColor || "#9013FE"

        if (!this.section.data.bg) {
            this.section.data.bg = wwLib.wwObject.getDefault({
                type: 'ww-color'
            });
            needUpdate = true
        }

        if (!this.section.data.features) {
            this.section.data.features = []
            needUpdate = true
        }
        if (!this.section.data.thumbnailsPerLine) {
            this.section.data.thumbnailsPerLine = 4;
            needUpdate = true;
        }

        if (_.isEmpty(this.section.data.features)) {
            for (let i = 0; i < 3; i++) {
                this.section.data.features.push(
                    {
                        background: wwLib.wwObject.getDefault({ type: 'ww-color', data: { color: 'white' } }),
                        contents: [],
                    },
                )
            }

            needUpdate = true;
        }

        if (_.isEmpty(this.section.data.topWwObjs)) {
            this.section.data.topWwObjs = [];
            needUpdate = true;
        }
        if (_.isEmpty(this.section.data.bottomWwObjs)) {
            this.section.data.bottomWwObjs = [];
            needUpdate = true;
        }

        if (needUpdate) {
            this.sectionCtrl.update(this.section);
        }
    },
    mounted() {
        this.init();
    },
    beforeDestroy() {
        window.removeEventListener("resize", this.setThumbnailsPerLine);
    },
    methods: {
        init() {
            this.setThumbnailsPerLine();
            window.addEventListener("resize", this.setThumbnailsPerLine);
        },
        nextSlide() {
            try {
                this.sliderPosition++
                if (this.sliderPosition > this.featuresLength - 1)
                    this.sliderPosition = 0
                this.$forceUpdate()
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        prevSlide() {
            try {
                this.sliderPosition--
                if (this.sliderPosition < 0)
                    this.sliderPosition = this.featuresLength - 1
                this.$forceUpdate()
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }

        },

        switchToIndex(index, position) {
            try {
                if (position < this.section.data.features.length && index != position) {
                    this.sliderPosition = position
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);

            }

        },

        setThumbnailsPerLine() {
            try {
                let width = window.innerWidth;

                if (width < 992) {
                    this.cardPercent = 90;
                }
                else {
                    this.cardPercent = 70;
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        /* wwManager:start */
        add(list, options) {
            try {
                list.splice(options.index, 0, options.wwObject);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },


        /* add a new section to the slider */
        addFeature(_index, where) {
            try {
                const up = (where == 'after') ? parseInt(1) : 0;
                const index = _index + up
                const newCard = getNewFeature()

                this.section.data.features.splice(index, 0, newCard);
                this.sectionCtrl.update(this.section);
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        removeFeature(index) {
            try {
                this.section.data.features.splice(index, 1);
                if (!this.section.data.features.length) {
                    this.addFeature(0, 'after');
                }
                this.sectionCtrl.update(this.section);
                this.prevSlide()
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);

            }
        },

        /* add picture */
        addElement(list, _index, where) {
            try {
                const up = (where == 'after') ? parseInt(1) : 0;
                const index = _index + up
                let newCopie = JSON.parse(JSON.stringify(list[0]))

                wwLib.wwUtils.changeUniqueIds(newCopie)
                list.splice(index, 0, newCopie);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        removeElement(list, index) {
            try {
                if (list.length > 1) {
                    list.splice(index, 1);
                    this.sectionCtrl.update(this.section);
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        async openOptions() {
            try {

                wwLib.wwPopups.addStory('WWSLIDER_CUSTOM', {
                    title: {
                        en: 'Dots options',
                        fr: 'Options des points de navigations'
                    },
                    type: 'wwPopupForm',
                    storyData: {
                        fields: [
                            {
                                label: {
                                    en: 'Navigation dots color:',
                                    fr: 'Couleur des points de navigations :'
                                },
                                type: 'color',
                                key: 'dotsColor',
                                valueData: 'section.data.dotColor',
                                desc: {
                                    en: 'Choose a Nav dots color:',
                                    fr: 'Changer la couleur des points de navigations :'
                                },
                            },

                        ]
                    },
                    buttons: {
                        NEXT: {
                            text: {
                                en: 'Finish',
                                fr: 'Terminer'
                            },
                            next: null
                        }
                    }
                })
                let options = {
                    firstPage: 'WWSLIDER_CUSTOM',
                    data: {
                        section: this.section,

                    },
                }

                const result = await wwLib.wwPopups.open(options)
                if (typeof (result) != 'undefined') {
                    if (typeof (result.dotsColor) != 'undefined') {
                        this.section.data.dotColor = result.dotsColor
                    }
                    this.sectionCtrl.update(this.section);
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        /* Used on mobile version to swipe */
        remove(list, options) {
            try {
                list.splice(options.index, 1);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }

        },

        async setFeatureImage(index) {
            try {
                let options = {
                    firstPage: 'IMAGE_SELECT'
                }

                const result = await wwLib.wwPopups.open(options)

                if (!result) {
                    return;
                }
                if (typeof (result.image) != 'undefined') {
                    this.section.data.features[index].preview = result.image;
                }

                this.sectionCtrl.update(this.section);
            }
            catch (error) {
                console.log(error);
            }

        }

        /* wwManager:end */



    }
};
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style lang='scss' scoped>
.section-container {
    margin: 0 10px;
    width: calc(100% - 20px);
    overflow-x: hidden;
    overflow-y: hidden;
    position: relative;

    @media (min-width: 992px) {
        margin: 0 20px;
        width: calc(100% - 40px);
    }
}

.background {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
}

.top-ww-objs,
.bottom-ww-objs {
    position: relative;
    .top-ww-obj,
    .bottom-ww-obj {
        position: relative;
    }
}

.full-container {
    position: relative;

    .fade {
        position: absolute;
        top: 0;
        bottom: 0;
        width: 100px;
        z-index: 1;
        display: none;
        cursor: pointer;
        pointer-events: all;
        transform: translateZ(0);
        &.right {
            right: 0;
        }
        &.left {
            left: 0;
        }

        @media (min-width: 992px) {
            display: block;
        }
    }

    .container {
        position: relative;
        pointer-events: all;
        @media (min-width: 768px) {
            width: 80%;
        }

        @media (min-width: 992px) {
            width: 90%;
        }

        .container-center {
            display: flex;
            transition: transform 0.5s ease;
            margin-left: 5%;
            @media (min-width: 1024px) {
                justify-content: center;
            }
            @media (min-width: 992px) {
                margin-left: 15%;
            }
            .thumbnail-container {
                width: 30%;
                position: relative;
                margin: 30px 0 30px 0px;
                min-height: 50px;
                transition: transform 0.4s ease-out, box-shadow 0.4s ease-out;
                display: flex;
                justify-content: center;

                .thumbnail {
                    $computer-width: 180%;

                    height: 100%;
                    width: 95%;
                    position: relative;

                    @media (min-width: 992px) {
                        width: 350px;
                    }
                    @media (min-width: 1200px) {
                        width: 400px;
                    }

                    .computer {
                        &.desktop {
                            position: absolute;
                            top: -15px;
                            left: 50%;
                            transform: translate(-50%, 0);
                            width: $computer-width;
                            padding-bottom: $computer-width * 0.581;
                            background-size: contain;
                            background-repeat: no-repeat;
                            display: none;

                            @media (min-width: 992px) {
                                display: block;
                            }

                            .website-preview {
                                position: absolute;
                                top: 4%;
                                left: 50%;
                                transform: translate(-50%, 0);
                                width: 76%;
                                padding-bottom: 48.26%;
                                overflow: hidden;

                                .preview {
                                    position: absolute;
                                    top: 0;
                                    left: 0;
                                    width: 100%;
                                    height: 100%;
                                    background-size: 100% auto;
                                    background-repeat: no-repeat;
                                }

                                &.scrolling {
                                    .preview {
                                        animation-name: scroll-website;
                                        animation-duration: 10s;
                                        animation-iteration-count: infinite;
                                    }
                                }
                            }
                        }

                        &.mobile {
                            width: 100%;

                            @media (min-width: 992px) {
                                display: none;
                            }

                            .website-preview {
                                width: calc(100% - 20px);
                                margin-left: 10px;
                                margin-top: 10px;
                                padding-bottom: 58.26%;
                                overflow: hidden;
                                position: relative;

                                .preview {
                                    position: absolute;
                                    top: 0;
                                    left: 0;
                                    width: 100%;
                                    height: 100%;
                                    background-size: 100% auto;
                                    background-repeat: no-repeat;
                                }

                                &.scrolling {
                                    .preview {
                                        animation-name: scroll-website;
                                        animation-duration: 10s;
                                        animation-iteration-count: infinite;
                                    }
                                }
                            }
                        }
                    }

                    .background {
                        border-radius: 7px;
                    }

                    .content {
                        position: relative;

                        @media (min-width: 992px) {
                            padding-top: $computer-width * 0.581;
                        }
                    }
                }
                /* wwManager:start */

                .contextmenu {
                    position: absolute;
                    top: 0;
                    left: 0;
                    width: 30px;
                    height: 30px;
                    color: white;
                    background-color: #ef811a;
                    border-radius: 100%;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    font-size: 1.2rem;
                    cursor: pointer;
                    z-index: 2;
                }
                .contextmenu-center {
                    left: calc(50% - 15px);
                }
                /* wwManager:end */
            }
        }
    }
}

@keyframes scroll-website {
    0% {
        background-position-y: 0%;
    }
    10% {
        background-position-y: 0%;
    }
    100% {
        background-position-y: 100%;
    }
}

.content-dots-wrapper {
    display: flex;
    list-style: none;
    justify-content: center;
    position: relative;
    padding-bottom: 15px;
    margin-top: 20px;

    .content-dot {
        margin-right: 15px;
        border-radius: 100%;
        border-width: 2px;
        border-style: solid;
        .dot {
            cursor: pointer;
            width: 15px;
            height: 15px;
            pointer-events: all;
        }
    }
}
</style>
