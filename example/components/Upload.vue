<template>
    <div class="ths_upload" ref="hookUpload" :class="[(!isVideo && myImgData && myImgData.imgUrl && 'th_active')||'']"
    :style="{'backgroundImage':!isVideo && (myImgData && myImgData.imgUrl && 'url(' + myImgData.imgUrl + '!200!200)') || ''}">
        <i class="th_delImg th_icon" v-if="myImgData && myImgData.imgUrl" @click="delImg($event)" @tap="delImg($event)">{{closeIcon}}</i>
    </div>
</template>

<script>
import { FileUpload } from 'h5-file-upload';
export default {
    name: 't-file-upload',
    model: {
        prop: 'imgData',
        event: 'valueChange'
    },
    props: {
        closeIcon: { // 关闭图标
            default: '\ue625'
        },
        imgFliter: { // imgUrl过滤器
            type: Function
        },
        imgData: { // 默认数据
        // default: {}
            default: function () {
                return {
                    imgUrl: 'https://avatar.csdn.net/9/F/E/3_phoebe_16.jpg'
                };
            }
        },
        uploadData: { // 上传时上送的数据
            type: Object
        },
        serviceCode: {
            type: String,
            default: ''
        },
        isVideo: { // 上传的是否为视频
            type: Boolean,
            default: false
        }
    },
    watch: {
        imgData (nowVal) { // 数据发生改变
            let _that = this;
            _that.myImgData = nowVal;
        },
        myImgData (nowVal) { // 数据发生改变
            let _that = this;
            _that.valueFilter(_that.myImgData);
            // _that.setBg(nowVal);
            _that.$emit('valueChange', nowVal);
        }
    },
    data () {
        return {
            myImgData: undefined
        };
    },
    mounted () {
        let _that = this;
        _that.$nextTick(() => {
            let el = _that.$refs.hookUpload;
            if (el) {
                if (_that.imgData) {
                    _that.myImgData = _that.imgData || _that.myImgData;
                }
                _that.inputFile = el;
                _that.initFileUploadOptions();
            }

            _that.initData = true;
            _that.initFileLoad();
        });
    },
    methods: {
        /**
         * 初始化图片上传参数
         *
         * @private
         * @memberof CommentComponent
         */
        initFileUploadOptions () {
            let _that = this;
            // let _options = _that._options;
            if (_that.serviceCode) {
                _that.uploadOptions = {
                    buttonText: '+',
                    formData: _that.uploadData || {},
                    uploader: _that.serviceCode,
                    onReaderFile: function (backData) {
                        let imgDataItem = _that.myImgData;
                        if (imgDataItem && imgDataItem.state !== 1) {
                            _that.$set(_that.myImgData, 'state', 1);
                        }
                        // _that.sLayer.toast('正在读取文件')
                        _that.$emit('onReaderFile', backData);
                    },
                    onCompressStart: function (backData) {
                        let imgDataItem = _that.myImgData;
                        if (imgDataItem && imgDataItem.state !== 1) {
                            _that.$set(_that.myImgData, 'state', 1);
                        }
                        // _that.sLayer.toast('正在压缩图片')
                        _that.$emit('onCompressStart', backData);
                    },
                    onSizeError: function (data) {
                        // _that.sLayer.toast('上传图片过大')

                        _that.$emit('onSizeError', data);
                    }, // 上传失败的动作
                    onFileTypeError: function (backData) {
                        _that.$emit('onFileTypeError', _that.myImgData);
                    }, // 上传失败的动作
                    onUploadStart: function (backData) {
                        let imgDataItem = _that.myImgData;
                        if (imgDataItem && imgDataItem.state !== 1) {
                            _that.$set(_that.myImgData, 'state', 1);
                        }
                        _that.$emit('onUploadStart', backData);
                    },
                    onInit: function (backData) {
                        let imgDataItem = _that.myImgData;
                        if (imgDataItem && backData) {
                            _that.$set(_that.myImgData, 'el', backData.el);
                        }
                        _that.$emit('onInit', _that.myImgData);
                    },
                    onUploadSuccess: function (backData, data) {
                        let imgDataItem = _that.myImgData;
                        if (typeof _that.layerIndex === 'number') {
                            _that.sLayer.close(_that.layerIndex);
                            _that.layerIndex = undefined;
                        }
                        if (data.code + '' === '00' || (data.code + '') === '0') {
                            if (imgDataItem) {
                                _that.myImgData = Object.assign({}, _that.myImgData, {
                                    data: data.result,
                                    state: 0
                                });
                            }
                            _that.$emit('onUploadSuccess', _that.myImgData);
                        }
                    },
                    onUploadError: function (backData) {
                        let imgDataItem = _that.myImgData;
                        if (typeof _that.layerIndex === 'number') {
                            _that.sLayer.close(_that.layerIndex);
                            _that.layerIndex = undefined;
                        }
                        if (imgDataItem && imgDataItem.state !== 0) {
                            _that.$set(_that.myImgData, 'state', 0);
                        }
                        // _that.sLayer.toast('上传失败')
                        _that.$emit('onUploadError', backData);
                    },
                    onUploadComplete: function (file, data) { // 上传结束后
                        _that.$emit('onUploadComplete', _that.myImgData);
                    }
                };
                if (_that.isVideo) {
                    _that.uploadOptions = Object.assign(_that.uploadOptions, { // 上传视频的参数
                        fileTypeExts: '*',
                        accept: 'video/*',
                        capture: '',
                        buttonText: '+'
                    });
                }
            }
        },
        /**
         * 删除图片
         *
         * @param {IImgDataItem} imgData 图片数据
         * @memberof CommentComponent
         */
        delImg (imgData) {
            let _that = this;
            _that.sLayer.confirm('确定要删除吗？', '系统消息', ['确定', '取消'],
                function (e) {
                    if (e.index === 0) {
                        _that.myImgData = {
                            el: _that.myImgData.el,
                            state: 0
                        };
                        _that.$emit('delImg', _that.myImgData);
                    }
                }
            );
        },
        /**
         * 上传后的值过滤器
         * @param {Object} data 数据
         */
        valueFilter (data) {
            let _that = this;
            if (_that.imgFliter instanceof Function) {
                let imgUrl = _that.imgFliter(data);
                if (data.imgUrl !== imgUrl) {
                    _that.$set(_that.myImgData, 'imgUrl', imgUrl);
                }
            }
        },

        /**
         * 初始化上传插件
         *
         * @private
         * @memberof FileUploadDirective
         */
        initFileLoad () {
            let _that = this;
            if (!_that.inputFile || _that.initEnd || !_that.initData || (!_that.uploadOptions && !_that.formData)) {
                return;
            }
            _that.initEnd = true;
            let def = {
                fileKey: _that.fileKey || '',
                auto: true,
                fileTypeExts: '*.jpg;*.png;*.jpeg;*.gif',
                accept: 'image/*', // 为空的时候为自动按fileTypeExts(文件扩展名)获取
                // capture: 'camera,camcorder,microphone',
                multi: false,
                dataType: 'json',
                fileSizeLimit: 1024 * 40, // kb
                buttonText: '+', // 上传按钮上的文字
                showUploadedPercent: false, // 是否实时显示上传的百分比，如20%
                showUploadedSize: false,
                removeTimeout: 100, // 上传完成后进度条的消失时间
                uploader: ''
            };
            let _options = Object.assign(def, _that.uploadOptions, _that.formData);

            new FileUpload(_that.inputFile, _options);
        }
    }
};

</script>

<style lang="scss">
    @import "../style/_public/_var";
    @import "../style/_public/_mixin";
    @import "../style/_public/_thFontIcon";
    //上传按钮栏
    .ths_upload-box{//上传框
        padding-top: 5px;
        display: block;
        width: 100%;
        font-size: 0;
        padding-bottom: 15px;
        background: $bgf;
        @include after();
    }
    .ths_upload{
        position: relative;
        display: inline-block;
        background-repeat: no-repeat;
        background-position: center center;
        background-size: cover;
        width: 55px;
        height: 52px;
        border: 1px solid $brc;
        // overflow: hidden; // 课程详情页面浮窗评论里不能hidden
        float: left;
        text-align: center;
        color:$color92;
        ._select-btn-text_{
            @extend %th_icon;
            font-size: 20px !important;
        }
        &.th_active{
            ._select-btn-text_{
                display: none !important;
            }
        }
        .th_text{
            float: left;
            display: inline-block;
            width: 100%;
            height: 100%;
            line-height: 52px;
            text-align: center;
            color: $color6;
            font-size: $fontSize28;
        }
        .th_delImg{
            // display: none;
            font-size: $fontSize24;
            width: 20px;
            height: 20px;
            line-height: 16px;
            text-align: center;
            position: absolute;
            right: 0px;
            top: 0px;
            color: $colorOrange;
            z-index: 99;
        }
    }
</style>
