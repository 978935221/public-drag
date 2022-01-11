<template>
  <div class="home">
    <el-input
      v-model="input"
      @blur="handleBlur"
      placeholder="请输入内容"
    ></el-input>
    <el-row>
      <el-button @click="sizebtn = 'mini'" type="primary">小图标</el-button>
      <el-button @click="sizebtn = 'middle'" type="success">大图标</el-button>
      <el-button @click="sizebtn = 'big'" type="info">超大图标</el-button>
      <el-button @click="sizebtn = 'tile'" type="warning">标准图标</el-button>
    </el-row>
    <div class="grid-body">
      <div
        class="grid-container"
        ref="trainSelected"
        id="gridParent"
        @mousedown.stop="_onLayoutMousedown"
      >
        <div
          class="grid-wrap"
          :id="'trans' + item.key"
          @mousedown="_onMousedown($event, item)"
          :draggable="draggable"
          v-for="item in layout"
          :style="{
            left: item.x + 'px',
            top: item.y + 'px',
            width: item.w + 'px',
            height: item.h + 'px',
            'z-index': moveItem.key === item.key ? 999 : 0,
          }"
          :key="item.key"
          :item="item"
        >
          <div :class="item.selected ? 'grid-wrap-select' : ''">
            {{ item.key }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import elementResizeDetectorMaker from "element-resize-detector";

export default {
  name: "Home",
  prop: {
    w: {
      type: String,
      default: "200",
    },
    h: {
      type: String,
      default: "200",
    },
    // sizebtn: {
    //   type: String,
    //   default: "middle",
    // },
  },
  watch: {
    sizebtn: function () {
      switch (this.sizebtn) {
        case "mini":
          // 80*45
          this.handleBoxHeight();
          this.size = {
            width: 100,
            height: 80,
          };
          this.layout.forEach((item) => {
            item.w = 100;
            item.h = 80;
          });
          this.handleBlur();
          break;
        case "middle":
          this.handleBoxHeight();
          // 160 * 90
          this.size = {
            width: 200,
            height: 150,
          };
          this.layout.forEach((item) => {
            item.w = 200;
            item.h = 150;
          });
          this.handleBlur();
          break;
        case "big":
          this.handleBoxHeight();
          // 240 * 135
          this.size = {
            width: 300,
            height: 200,
          };
          this.layout.forEach((item) => {
            item.w = 300;
            item.h = 200;
          });
          this.handleBlur();
          break;
        case "tile":
          this.handleBoxHeight();
          // 80*45
          this.size = {
            width: 100,
            height: 80,
          };
          this.layout.forEach((item) => {
            item.w = 100;
            item.h = 80;
          });
          this.handleBlur();
          break;
      }
    },
  },
  data: function () {
    return {
      sizebtn: "mini",
      layout: [
        {
          i: "172.1.1.0",
          key: "172.1.1.0",
          x: 0,
          y: 30,
          w: 200,
          h: 150,
          selected: false,
        },
        {
          i: "172.1.1.1",
          key: "172.1.1.1",
          x: 200,
          y: 30,
          w: 200,
          h: 150,
          selected: false,
        },
        {
          i: "172.1.1.2",
          key: "172.1.1.2",
          x: 400,
          y: 30,
          w: 200,
          h: 150,
          selected: false,
        },
        {
          i: "172.1.1.3",
          key: "172.1.1.3",
          x: 600,
          y: 30,
          w: 200,
          h: 150,
          selected: false,
        },
        {
          i: "172.1.1.4",
          key: "172.1.1.4",
          x: 800,
          y: 30,
          w: 200,
          h: 150,
          selected: false,
        },
        {
          i: "172.1.1.5",
          key: "172.1.1.5",
          x: 1000,
          y: 30,
          w: 200,
          h: 150,
          selected: false,
        },
        {
          i: "172.1.1.6",
          key: "172.1.1.6",
          x: 1200,
          y: 30,
          w: 200,
          h: 150,
          selected: false,
        },
      ],
      draggable: true,
      input: "20",
      size: {
        width: 200,
        height: 150,
      },
      isMoseDown: false,
      moveItem: {},
      selectedState: [],
      boxWidth: 0,
      dragitem: {
        x: 0,
        y: 0,
      },
      dragData: {
        start: false,
        x: 0,
        y: 0,
        x0: 0,
        y0: 0,
        rectEl: null,
      },
      start: {
        x: null,
        y: null,
        x0: null,
        y0: null,
      },
    };
  },
  mounted() {
    this.handleBoxWidth();
    if (this.layout.length > 0) {
      this.handleBoxHeight();
    }
    window.addEventListener("dragstart", (e) => {
      e.preventDefault();
      e.stopPropagation();
    });
    document.addEventListener("mousemove", this._onMousemove);
    document.addEventListener("mouseup", this._onMouseup);
  },
  destroyed() {
    document.removeEventListener("mousemove", this._onMousemove);
    document.removeEventListener("mouseup", this._onMouseup);
    const { rectEl } = this.dragData;
    if (!rectEl) {
      return;
    }
    rectEl.parentNode.removeChild(rectEl);
    this.dragData.rectEl = null;
  },
  methods: {
    handleBoxHeight() {
      let arr = this.layout.map((item) => {
        return item.y;
      });
      document.getElementById("gridParent").style.height =
        Math.max(...arr) + this.layout[0].h > 250
          ? Math.max(...arr) + this.layout[0].h
          : 250 + "px";
    },
    handleBoxWidth() {
      const _this = this;
      const erd = elementResizeDetectorMaker();
      erd.listenTo(document.querySelector("#gridParent"), (element) => {
        // 这里的this.$refs.fan指定要监听的元素对象，对应的是<div ref="fan"></div>
        const width = element.offsetWidth;
        _this.$nextTick(() => {
          // 这里填写监听改变后的操作
          this.boxWidth = width;
        });
      });
    },
    isRectIntersect(rect1, rect2, ignorePixels = 5) {
      if (
        rect1.x > rect2.x + rect2.w - ignorePixels ||
        rect2.x > rect1.x + rect1.w - ignorePixels
      ) {
        return false;
      }
      if (
        rect1.y > rect2.y + rect2.h - ignorePixels ||
        rect2.y > rect1.y + rect1.h - ignorePixels
      ) {
        return false;
      }
      return true;
    },
    _eachGridItem(fn) {
      const idmap = this.layout.reduce((M, item) => {
        M[item.i] = item;
        return M;
      }, {});
      this.$refs.trainSelected.childNodes.forEach((childVm) => {
        if (childVm.className === "grid-wrap") {
          fn(childVm, idmap[childVm.id.split("trans")[1]]);
        }
      });
    },
    updateSelection(selection) {
      this._eachGridItem((el, layout) => {
        const innerRect = el.getBoundingClientRect();
        const rect = {
          x: parseInt(el.style.left),
          y: parseInt(el.style.top),
          w: innerRect.width,
          h: innerRect.height,
        };
        layout.selected = this.isRectIntersect(selection, rect);
      });
      this.selectedState = [];
    },
    handleBlur() {
      if (this.input === "") {
        return;
      }
      if (Number(this.input) > parseInt(this.boxWidth / this.size.width)) {
        this.input = parseInt(this.boxWidth / this.size.width).toString();
      }
      let sum = 1;
      // parseInt(this.boxWidth / this.size.width)
      for (let i = 0; i < this.layout.length; i++) {
        for (let j = 0; j < Number(this.input); j++) {
          this.layout[sum - 1].x = j * this.size.width;
          this.layout[sum - 1].y = i * this.size.height;
          console.log(i, j);
          sum++;
          // console.log(i * this.size.width, j * this.size.height);
          if (sum > this.layout.length) {
            return;
          }
        }
      }

      // if (this.input > 0) {
      // }
    },
    // 当前鼠标到盒子的距离
    getElementXY(e) {
      const rect = e.target.getBoundingClientRect();
      const x = e.clientX - rect.left; // x position within the element.
      const y = e.clientY - rect.top; // y position within the element.
      return { x, y };
    },
    _onLayoutMousedown(e) {
      console.log(e);
      const layoutEl = document.getElementById("gridParent");
      if (e.target !== layoutEl) {
        return;
      }
      this.selectedState = [];
      const { dragData } = this;
      const { x: posx, y: posy } = this.getElementXY(e);
      let rectEl = dragData.rectEl;
      if (!rectEl) {
        rectEl = dragData.rectEl = document.createElement("div");
        rectEl.className = "temp-div";
        rectEl.style.border = "1px solid #000";
        rectEl.style.position = "absolute";
        rectEl.style.userSelect = "none";
        layoutEl.appendChild(rectEl);
      }
      dragData.x = posx;
      dragData.y = posy;
      dragData.x0 = e.pageX;
      dragData.y0 = e.pageY;
      rectEl.style.left = posx + "px";
      rectEl.style.top = posy + "px";
      rectEl.style.width = "0px";
      rectEl.style.height = "0px";
      rectEl.style.display = "block";

      dragData.start = true;
      this.layout.forEach((item) => {
        item.selected = false;
      });
    },
    _onMousedown(e, item) {
      this.moveItem = item;
      this.isMoseDown = true;
      const wrap = document.getElementById(`trans${item.key}`);
      let x = e.pageX - wrap.offsetLeft;
      let y = e.pageY - wrap.offsetTop;
      const { dragitem } = this;
      dragitem.x = x;
      dragitem.y = y;
      const { x: posx, y: posy } = this.getElementXY(e);
      this.start.x = posx;
      this.start.y = posy;
      this.start.top = item.y;
      this.start.left = item.x;
    },
    _onMousemove(e) {
      const { dragitem, dragData } = this;
      const { x, y } = dragitem;

      if (dragData.start) {
        const { x: posx, y: posy, x0, y0, rectEl } = dragData;
        const x1 = posx + e.pageX - x0;
        const y1 = posy + e.pageY - y0;
        const rect = {
          x: Math.min(x1, posx),
          y: Math.min(y1, posy),
          w: Math.abs(e.pageX - x0),
          h: Math.abs(e.pageY - y0),
        };
        rectEl.style.left = rect.x + "px";
        rectEl.style.top = rect.y + "px";
        rectEl.style.width = rect.w + "px";
        rectEl.style.height = rect.h + "px";

        this.updateSelection(rect);
      }

      if (this.isMoseDown) {
        const wrap = document.getElementById(`trans${this.moveItem.key}`);
        const parent = document.getElementById("gridParent");
        let xx = e.pageX - x;
        let yy = e.pageY - y;
        // 设置边界限制
        xx = xx < 0 ? 0 : xx;
        yy = yy < 0 ? 0 : yy;
        if (xx >= innerWidth - wrap.offsetWidth) {
          document.documentElement.scrollLeft = 20;
        } else {
          document.documentElement.scrollLeft = 0;
        }
        xx =
          xx > parent.offsetWidth - wrap.offsetWidth
            ? parent.offsetWidth - wrap.offsetWidth
            : xx;
        this.moveItem.x = xx;
        this.moveItem.y = yy;
        // 禁止X滚动轴
        document.body.style.overflowX = "hidden";
        wrap.style.marginLeft = 0;
        wrap.style.marginTop = 0;
      }
    },
    _onMouseup() {
      const { start, dragData } = this;
      const { x: posx, y: posy, left: left, top: top } = start;
      // 若发生重叠现象则进行替换
      if (this.isMoseDown) {
        this.layout.forEach((item) => {
          if (
            this.moveItem.key !== item.key &&
            posx + this.moveItem.x > item.x &&
            posx + this.moveItem.x < item.x + item.w &&
            posy + this.moveItem.y > item.y &&
            posy + this.moveItem.y < item.y + item.h
          ) {
            const x = item.x;
            const y = item.y;
            item.x = left;
            item.y = top;
            this.moveItem.x = x;
            this.moveItem.y = y;
          }
        });
      }
      this.moveItem = {};
      this.start = {
        posx: null,
        posy: null,
        left: null,
        top: null,
      };
      this.isMoseDown = false;
      this.handleBoxHeight();
      if (!dragData.start) {
        return;
      }
      dragData.start = false;
      dragData.rectEl.style.display = "none";
      this.pressed = false;
    },
  },
};
</script>

<style scoped>
.grid-body {
  overflow: auto;
  height: 250px;
}

.grid-container {
  position: relative;
}

.grid-wrap {
  user-select: none;
  position: absolute;
  text-align: center;
  border: 2px solid #e5e4e9;
  box-sizing: border-box;
  background-color: #f68f26;
}

.grid-wrap-select {
  border: 2px solid #911717;
}

.item .item-body {
  box-sizing: border-box;
  user-select: none;
}

.temp-div {
  border: 5px dashed blue;
  background: #5a72f8;
  position: absolute;
  width: 0;
  height: 0;
}

::-webkit-scrollbar {
  width: 5px;
  /* 滚动条宽度， width：对应竖滚动条的宽度  height：对应横滚动条的高度*/
  height: 5px;
  background: rgb(255, 255, 255);
}

/*定义滚动条轨道（凹槽）样式*/
::-webkit-scrollbar-track {
  -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
  /* 较少使用 */
  border-radius: 3px;
}

/*定义滑块 样式*/
::-webkit-scrollbar-thumb {
  border-radius: 3px;
  height: 100px;
  /* 滚动条滑块长度 */
  background-color: rgb(133, 133, 133);
}
</style>
