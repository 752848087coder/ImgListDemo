<template>
  <div id="app">
        <div
        class="row"
        v-for="(row, index) in showImgs"
        :key="index" >
            <div
            v-for="(img, key) in row"
            :key="key"
            class ="item"
            :style="`background-image: url(${img.src});width: ${img.sw}px;height:${img.sh}px`"
            @click="showPhoto(img.src)"
            />
        </div>
        <div
        @click="closePhoto"
        :style="`background-image: url(${curSrc});`"
        v-if="curSrc"
        class="big-photo"
        >
        </div>
  </div>
</template>

<script>
export default {
    name: "App",
    data() {
        return {
            imgs: [],
            showImgs: [],
            curSrc: '',
            data: [],
            scroll: undefined
        }
    },
    methods: {
        initImgs() {
            let promises = [];
            for (let i = 1; i <= 20; i++) {
                promises.push(
                    new Promise((resolve, reject) => {
                        let img = new Image();
                        img.src = `http://gavincat.cn:8080/${i}.jpg`;
                        img.onload = () => {
                            this.data.push({
                                src: img.src,
                                ratio: img.width / img.height,
                                w: 200 *　img.width / img.height,
                                h: 200
                            });
                            resolve();
                        }
                    })
                );
            }
            Promise.all(promises).then(() => {
                this.imgs = this.data.map(i => {
                    return {
                        src: i.src,
                        ratio: i.ratio,
                        w: i.w,
                        h: i.h
                    }
                });
                this.handleImgs();
            });
        },
        handleImgs() {
            this.showImgs = [];
            let clientWidth = document.body.clientWidth - 2 * 5;
            let imgs = this.imgs;
            let count = 0;
            let row = [];
            imgs.forEach(img => {
                count += img.w;
                row.push(img);
                if(count > clientWidth) {
                    let last = clientWidth - count;
                    let length = row.length;
                    row.forEach(i => {
                        if (i.handled) return;
                        i.sw = i.w - last * i.w / (clientWidth - (length - 1) * 5);
                        i.sh = i.h;
                        i.handled = true;
                    });
                    this.showImgs.push(row);
                    row = [];
                    count = 0;
                }
            })
            if (row.length > 0) {
                this.showImgs.push(row);
            }
            // 每次渲染完数据调用一下scroll，如果距离底部距离过大会继续自动拉取更多图片
            this.scroll();
        },
        fetchData() {
            this.imgs.push(...this.data.map(i => {
                return {
                    src: i.src,
                    ratio: i.ratio,
                    w: i.w,
                    h: i.h
                }
            }));
        },
        showPhoto(src) {
            this.curSrc = src;
        },
        closePhoto() {
            this.curSrc = '';
        }
    },
    created() {
        let resize = () => {
            let timeout = null;
            return (e) => {
                if(timeout) return;
                timeout = setTimeout(() => {
                    this.imgs.forEach(i => i.handled = false);
                    this.handleImgs();
                    timeout = null;
                }, 300);
            }
        };
        let scroll = () => {
            let timeout = null;
            return (e) => {
                if(timeout) return;
                timeout = setTimeout(() => {
                    let clientHeight = document.documentElement.clientHeight;
                    let scrollTop = document.documentElement.scrollTop;
                    let scrollHeight = document.documentElement.scrollHeight;
                    if (scrollHeight - (scrollTop + clientHeight) < 200) {
                        this.fetchData();
                        this.handleImgs();
                    }
                    timeout = null;
                }, 300);
            }
        }
        window.onresize = resize();
        this.scroll = window.onscroll = scroll();
        this.initImgs();
    }
};
</script>

<style>
html{
    min-width: 1000px;
}
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding: 5px;
}
.row{
    margin-bottom: 5px;
    display: flex;
}
.item {
    background-position: center;
    background-size: cover;
    height: 200px;
    margin-right: 5px;
    cursor: zoom-in;
}
.row :nth-last-child(1){
    margin-right: 0px; 
}
.big-photo{
    background-size: contain;
    background: rgba(0, 0, 0, 0.9);
    background-repeat: no-repeat;
    background-position: center;
    position: fixed;
    top:0;
    left: 0;
    right: 0;
    bottom: 0;
    cursor: zoom-out;
}
</style>
