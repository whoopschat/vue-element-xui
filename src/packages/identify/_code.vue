<template>
  <canvas
    ref="canvas"
    @click="click"
    :width="contentWidth"
    :height="contentHeight"
  ></canvas>
</template>

<script>
export default {
  props: {
    identifyCode: {
      type: String,
      default: "1234",
    },
    fontSizeMin: {
      type: Number,
      default: 18,
    },
    fontSizeMax: {
      type: Number,
      default: 40,
    },
    backgroundColorMin: {
      type: Number,
      default: 180,
    },
    backgroundColorMax: {
      type: Number,
      default: 240,
    },
    colorMin: {
      type: Number,
      default: 50,
    },
    colorMax: {
      type: Number,
      default: 160,
    },
    lineColorMin: {
      type: Number,
      default: 40,
    },
    lineColorMax: {
      type: Number,
      default: 180,
    },
    dotColorMin: {
      type: Number,
      default: 0,
    },
    dotColorMax: {
      type: Number,
      default: 255,
    },
    contentWidth: {
      type: Number,
      default: 111,
    },
    contentHeight: {
      type: Number,
      default: 38,
    },
  },
  methods: {
    randomNum(min, max) {
      return Math.floor(Math.random() * (max - min) + min);
    },
    randomColor(min, max) {
      let r = this.randomNum(min, max);
      let g = this.randomNum(min, max);
      let b = this.randomNum(min, max);
      return "rgb(" + r + "," + g + "," + b + ")";
    },
    drawPic() {
      let canvas = this.$refs.canvas;
      let ctx = canvas.getContext("2d");
      ctx.textBaseline = "bottom";
      ctx.fillStyle = this.randomColor(
        this.backgroundColorMin,
        this.backgroundColorMax
      );
      ctx.fillRect(0, 0, this.contentWidth, this.contentHeight);
      for (let i = 0; i < this.identifyCode.length; i++) {
        this.drawText(ctx, this.identifyCode[i], i);
      }
    },
    drawText(ctx, txt, i) {
      ctx.fillStyle = this.randomColor(this.colorMin, this.colorMax);
      ctx.font =
        this.randomNum(this.fontSizeMin, this.fontSizeMax) + "px SimHei";
      let x = (i + 1) * (this.contentWidth / (this.identifyCode.length + 1));
      let y = this.randomNum(this.fontSizeMax, this.contentHeight - 5);
      var deg = this.randomNum(-30, 30);
      ctx.translate(x, y);
      ctx.rotate((deg * Math.PI) / 180);
      ctx.fillText(txt, 0, 0);
      ctx.rotate((-deg * Math.PI) / 180);
      ctx.translate(-x, -y);
    },
    drawLine(ctx) {
      for (let i = 0; i < 8; i++) {
        ctx.strokeStyle = this.randomColor(
          this.lineColorMin,
          this.lineColorMax
        );
        ctx.beginPath();
        ctx.moveTo(
          this.randomNum(0, this.contentWidth),
          this.randomNum(0, this.contentHeight)
        );
        ctx.lineTo(
          this.randomNum(0, this.contentWidth),
          this.randomNum(0, this.contentHeight)
        );
        ctx.stroke();
      }
    },
    drawDot(ctx) {
      for (let i = 0; i < 100; i++) {
        ctx.fillStyle = this.randomColor(0, 255);
        ctx.beginPath();
        ctx.arc(
          this.randomNum(0, this.contentWidth),
          this.randomNum(0, this.contentHeight),
          1,
          0,
          2 * Math.PI
        );
        ctx.fill();
      }
    },
    click() {
      this.$emit("click");
    },
  },
  watch: {
    identifyCode() {
      this.drawPic();
    },
  },
  mounted() {
    this.drawPic();
  },
};
</script>