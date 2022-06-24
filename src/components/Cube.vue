<template>
  <div id="cube" :style="{
    '--width': cube.layer * cube.size,
    '--height': cube.layer * cube.size,
    '--size': cube.size,
    '--rotate-x': cube.rotates.x,
    '--rotate-y': cube.rotates.y,
    '--rotate-z': cube.rotates.z,
  }">
    <template v-for="x in cube.layer">
      <template v-for="y in cube.layer">
        <v-touch class="block" :x="x - 1" :y="y - 1" :z="z - 1" v-for="z in cube.layer" :key="'' + x + y + z" :style="{
          '--rotate-x': blocks[x - 1][y - 1][z - 1].rotates.x,
          '--rotate-y': blocks[x - 1][y - 1][z - 1].rotates.y,
          '--rotate-z': blocks[x - 1][y - 1][z - 1].rotates.z,
          '--translate-x': blocks[x - 1][y - 1][z - 1].translates.x,
          '--translate-y': blocks[x - 1][y - 1][z - 1].translates.y,
          '--translate-z': blocks[x - 1][y - 1][z - 1].translates.z,
          transition: animation.enable
            ? `transform ${animation.duration / 1000}s ease, -webkit-transform ${animation.duration / 1000}s ease`
            : '',
        }">
          <v-touch v-for="(side, index) in sides" :key="'' + x + y + z + index" :class="[side, 'side']" :style="{
            '--color': blocks[x - 1][y - 1][z - 1].colors[side],
          }" @swipeleft="swipeleft(x - 1, y - 1, z - 1, side)" @swiperight="swiperight(x - 1, y - 1, z - 1, side)"
            @swipeup="swipeup(x - 1, y - 1, z - 1, side)" @swipedown="swipedown(x - 1, y - 1, z - 1, side)"></v-touch>
        </v-touch>
      </template>
    </template>
  </div>
</template>

<script>
export default {
  name: 'cube',
  data() {
    return {
      sides: ['left', 'right', 'top', 'bottom', 'front', 'back'],
      animation: {
        enable: false,
        duration: 500,
      },
      cube: {
        layer: 3,
        size: 100,
        colors: {
          ground: '#000000',
          left: '#0046ad',
          right: '#009b48',
          top: '#ffd500',
          bottom: '#cecece',
          front: '#b71234',
          back: '#ff5800',
        },
        rotates: { x: -30, y: 130, z: 0 },
      },
      blocks: [],
    };
  },
  computed: {
    perspective() {
      let config = { left: '', right: '', top: '', bottom: '' };

      function resetPerspectiveY(config, x, y) {
        y = (y % 360 < 0) ? (y % 360 + 360) : (y % 360);

        if (y > 0 && y <= 90) {
          config.left = 'left';
          config.right = 'front';
        } else if (y > 90 && y <= 180) {
          config.left = 'back';
          config.right = 'left';
        } else if (y > 180 && y <= 270) {
          config.left = 'right';
          config.right = 'back';
        } else {
          config.left = 'front';
          config.right = 'right';
        }
      }

      function resetPerspectiveX(config, x, y) {
        let temp;
        x = (x % 360 < 0) ? (x % 360 + 360) : (x % 360);

        if (x > 0 && x <= 90) {
          resetPerspectiveY(config, x, y);
          config.bottom = 'bottom';

        } else if (x > 90 && x <= 180) {
          resetPerspectiveY(config, x, y + 180);
          config.top = 'bottom';
          temp = config.left;
          config.left = config.right;
          config.right = temp;
        } else if (x > 180 && x <= 270) {
          resetPerspectiveY(config, x, y + 180);
          config.bottom = 'top';
          temp = config.left;
          config.left = config.right;
          config.right = temp;
        } else {
          resetPerspectiveY(config, x, y);
          config.top = 'top';
        }
      }

      resetPerspectiveX(config, this.cube.rotates.x, this.cube.rotates.y)
      return config;
    }
  },
  created() {
    this.initBlocks();
  },
  mounted() {
    console.log(this.perspective);
    var timer = null;
    document.onkeyup = (e) => {
      e = e || event || window.event || arguments.callee.caller.arguments[0];
      // if (timer || !e) return;
      this.controller(e.key);
      // timer = setTimeout(() => {
      //   timer = null;
      // }, this.animation.duration + 20);
    };
  },
  destroyed() {
    document.onkeyup = null;
  },
  methods: {
    initBlocks() {
      for (let x = 0; x < this.cube.layer; x++) {
        this.$set(this.blocks, x, new Array());
        for (let y = 0; y < this.cube.layer; y++) {
          this.$set(this.blocks[x], y, new Array());
          for (let z = 0; z < this.cube.layer; z++) {
            this.$set(this.blocks[x][y], z, {
              x: x,
              y: y,
              z: z,
              translates: {
                x: x * this.cube.size - (this.cube.layer * this.cube.size) / 2,
                y: y * this.cube.size - (this.cube.layer * this.cube.size) / 2,
                z: z * -this.cube.size + (this.cube.layer * this.cube.size) / 2,
              },
              rotates: { x: 0, y: 0, z: 0 },
              colors: {
                left: x == 0 ? this.cube.colors.left : this.cube.colors.ground,
                right:
                  x == this.cube.layer - 1
                    ? this.cube.colors.right
                    : this.cube.colors.ground,
                top: y == 0 ? this.cube.colors.top : this.cube.colors.ground,
                bottom:
                  y == this.cube.layer - 1
                    ? this.cube.colors.bottom
                    : this.cube.colors.ground,
                front:
                  z == 0 ? this.cube.colors.front : this.cube.colors.ground,
                back:
                  z == this.cube.layer - 1
                    ? this.cube.colors.back
                    : this.cube.colors.ground,
              },
            });
          }
        }
      }
    },
    controller(action) {
      const speed = 10;
      switch (action) {
        case 'ArrowUp':
          this.cube.rotates.x += speed;
          break;
        case 'ArrowDown':
          this.cube.rotates.x -= speed;
          break;
        case 'ArrowLeft':
          this.cube.rotates.y -= speed;
          break;
        case 'ArrowRight':
          this.cube.rotates.y += speed;
          break;
        case 'f':
          this.rotateZ(0, 90);
          break;
        case 'F':
          this.rotateZ(0, -90);
          break;
        case 'l':
          this.rotateX(0, 90);
          break;
        case 'L':
          this.rotateX(0, -90);
          break;
        case 'r':
          this.rotateX(this.cube.layer - 1, 90);
          break;
        case 'R':
          this.rotateX(this.cube.layer - 1, -90);
          break;
        case 'u':
          this.rotateY(0, 90);
          break;
        case 'U':
          this.rotateY(0, -90);
          break;
        case 'd':
          this.rotateY(this.cube.layer - 1, 90);
          break;
        case 'D':
          this.rotateY(this.cube.layer - 1, -90);
          break;
      }
    },
    swipeleft(x, y, z, side) {
      this._swipeHorizontal(x, y, z, side, -90);
    },
    swiperight(x, y, z, side) {
      this._swipeHorizontal(x, y, z, side, +90);
    },
    swipeup(x, y, z, side) {
      this._swipeVertical(x, y, z, side, -90);
    },
    swipedown(x, y, z, side) {
      this._swipeVertical(x, y, z, side, +90);
    },
    _swipeHorizontal(x, y, z, side, degree) {
      let { left, right, top, bottom } = this.perspective;
      if (top == 'bottom' || bottom == 'top')
        degree = -degree;
      this.rotateY(y, degree);
    },
    _swipeVertical(x, y, z, side, degree) {
      let { left, right, top, bottom } = this.perspective;

      if (this.cube.rotates.y >= 0 && this.cube.rotates.y <= 180)
        degree = -degree

      if (side == this.perspective.left) {
        if ((top == 'top' || bottom == 'bottom') && (side == 'back' || side == 'front'))
          degree = -degree;
      } else if (side == this.perspective.right) {
        if ((top == 'bottom' || bottom == 'top') && (side == 'back' || side == 'front'))
          degree = -degree;
      } else return;

      if (side == 'front' || side == 'back')
        this.rotateX(x, degree);
      if (side == 'left' || side == 'right')
        this.rotateZ(z, degree);
    },
    rotateX(x, degree) {
      if (this.animation.enable) return;
      this.animation.enable = true;

      for (let y = 0; y < this.cube.layer; y++) {
        for (let z = 0; z < this.cube.layer; z++) {
          this.blocks[x][y][z].rotates.x += degree;
        }
      }

      setTimeout(() => {
        for (let y = 0; y < this.cube.layer; y++) {
          for (let z = 0; z < this.cube.layer; z++) {
            this.animation.enable = false;
            this.blocks[x][y][z].rotates.x = 0;
          }
        }
        this._rotateX(x, degree);
      }, this.animation.duration + 10);
    },
    _rotateX(x, orientation) {
      let cache = this.getCache();

      for (let y = 0; y < this.cube.layer; y++) {
        for (let z = 0; z < this.cube.layer; z++) {
          let colors = this.blocks[x][y][z].colors;

          let _colors =
            orientation > 0
              ? cache[x][this.cube.layer - 1 - z][y].colors ||
              this.blocks[x][this.cube.layer - 1 - z][y].colors
              : cache[x][z][this.cube.layer - 1 - y].colors ||
              this.blocks[x][z][this.cube.layer - 1 - y].colors;

          cache[x][y][z].colors = JSON.parse(JSON.stringify(colors));
          colors.left = _colors.left;
          colors.right = _colors.right;

          if (orientation > 0) {
            colors.back = _colors.top;
            colors.top = _colors.front;
            colors.bottom = _colors.back;
            colors.front = _colors.bottom;
          } else {
            colors.back = _colors.bottom;
            colors.top = _colors.back;
            colors.bottom = _colors.front;
            colors.front = _colors.top;
          }
        }
      }
    },
    rotateY(y, degree) {
      if (this.animation.enable) return;
      this.animation.enable = true;

      for (let x = 0; x < this.cube.layer; x++) {
        for (let z = 0; z < this.cube.layer; z++) {
          this.blocks[x][y][z].rotates.y += degree;
        }
      }

      setTimeout(() => {
        for (let x = 0; x < this.cube.layer; x++) {
          for (let z = 0; z < this.cube.layer; z++) {
            this.animation.enable = false;
            this.blocks[x][y][z].rotates.y = 0;
          }
        }

        let times = Math.abs(degree / 90);
        for (let time = 0; time < times; time++)
          this._rotateY(y, (degree > 0) ? true : false);
      }, this.animation.duration + 10);
    },
    _rotateY(y, orientation) {
      let cache = this.getCache();

      for (let x = 0; x < this.cube.layer; x++) {
        for (let z = 0; z < this.cube.layer; z++) {
          let colors = this.blocks[x][y][z].colors;

          let _colors =
            orientation > 0
              ? cache[z][y][this.cube.layer - 1 - x].colors ||
              this.blocks[z][y][this.cube.layer - 1 - x].colors
              : cache[this.cube.layer - 1 - z][y][x].colors ||
              this.blocks[this.cube.layer - 1 - z][y][x].colors;
          cache[x][y][z].colors = JSON.parse(JSON.stringify(colors));

          colors.top = _colors.top;
          colors.bottom = _colors.bottom;
          if (orientation > 0) {
            colors.back = _colors.right;
            colors.left = _colors.back;
            colors.front = _colors.left;
            colors.right = _colors.front;
          } else {
            colors.front = _colors.right;
            colors.left = _colors.front;
            colors.back = _colors.left;
            colors.right = _colors.back;
          }
        }
      }
    },
    rotateZ(z, degree) {
      if (this.animation.enable) return;
      this.animation.enable = true;

      for (let x = 0; x < this.cube.layer; x++) {
        for (let y = 0; y < this.cube.layer; y++) {
          this.blocks[x][y][z].rotates.z += degree;
        }
      }

      setTimeout(() => {
        for (let x = 0; x < this.cube.layer; x++) {
          for (let y = 0; y < this.cube.layer; y++) {
            this.animation.enable = false;
            this.blocks[x][y][z].rotates.z = 0;
          }
        }

        let times = Math.abs(degree / 90);
        for (let time = 0; time < times; time++)
          this._rotateZ(z, (degree > 0) ? true : false);
      }, this.animation.duration + 10);
    },
    _rotateZ(z, orientation) {
      let cache = this.getCache();

      for (let x = 0; x < this.cube.layer; x++) {
        for (let y = 0; y < this.cube.layer; y++) {
          let colors = this.blocks[x][y][z].colors;
          let _colors =
            orientation > 0
              ? cache[y][this.cube.layer - 1 - x][z].colors ||
              this.blocks[y][this.cube.layer - 1 - x][z].colors
              : cache[this.cube.layer - 1 - y][x][z].colors ||
              this.blocks[this.cube.layer - 1 - y][x][z].colors;

          cache[x][y][z].colors = JSON.parse(JSON.stringify(colors));
          colors.front = _colors.front;
          colors.back = _colors.back;

          if (orientation > 0) {
            colors.left = _colors.bottom;
            colors.bottom = _colors.right;
            colors.right = _colors.top;
            colors.top = _colors.left;
          } else {
            colors.left = _colors.top;
            colors.top = _colors.right;
            colors.bottom = _colors.left;
            colors.right = _colors.bottom;
          }
        }
      }
    },
    getCache() {
      let cache = [];
      for (let x = 0; x < this.cube.layer; x++) {
        cache[x] = [];
        for (let y = 0; y < this.cube.layer; y++) {
          cache[x][y] = [];
          for (let z = 0; z < this.cube.layer; z++) {
            cache[x][y][z] = {};
          }
        }
      }
      return cache;
    },
  },
};
</script>

<style scoped>
#cube {
  position: relative;
  margin: auto;
  width: calc(var(--width) * 1px);
  height: calc(var(--height) * 1px);
  box-sizing: border-box;
  transform: rotateX(calc(var(--rotate-x, 0) * 1deg)) rotateY(calc(var(--rotate-y, 0) * 1deg)) rotateZ(calc(var(--rotate-z, 0) * 1deg));
  transform-style: preserve-3d;
  transition: -webkit-transform 1s ease;
  transition: transform 1s ease;
  transition: transform 1s ease, -webkit-transform 1s ease;
}
.block {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  font-size: 0;
  border-radius: 12px;
  box-sizing: border-box;
  transform: rotateX(calc(var(--rotate-x, 0) * 1deg)) rotateY(calc(var(--rotate-y, 0) * 1deg)) rotateZ(calc(var(--rotate-z, 0) * 1deg));
  transform-style: preserve-3d;
}
.side {
  position: absolute;
  top: 0;
  left: 0;
  width: calc(var(--size) * 1px);
  height: calc(var(--size) * 1px);
  border: 6px solid black;
  border-radius: 12px;
  box-sizing: border-box;
  background-color: black;
}
.left {
  transform-origin: left;
  transform: translateX(calc(var(--translate-x, 0) * 1px)) translateY(calc(var(--translate-y, 0) * 1px)) translateZ(calc(var(--translate-z, 0) * 1px)) rotateY(90deg);
  background-color: var(--color, black);
}
.right {
  transform-origin: right;
  transform: translateX(calc(var(--translate-x, 0) * 1px)) translateY(calc(var(--translate-y, 0) * 1px)) translateZ(calc(var(--translate-z, 0) * 1px)) rotateY(-90deg);
  background-color: var(--color, black);
}
.top {
  transform-origin: top;
  transform: translateX(calc(var(--translate-x, 0) * 1px)) translateY(calc(var(--translate-y, 0) * 1px)) translateZ(calc(var(--translate-z, 0) * 1px)) rotateX(-90deg);
  background-color: var(--color, black);
}
.bottom {
  transform-origin: bottom;
  transform: translateX(calc(var(--translate-x, 0) * 1px)) translateY(calc(var(--translate-y, 0) * 1px)) translateZ(calc(var(--translate-z, 0) * 1px)) rotateX(90deg);
  background-color: var(--color, black);
}
.front {
  transform: translateX(calc(var(--translate-x, 0) * 1px)) translateY(calc(var(--translate-y, 0) * 1px)) translateZ(calc(var(--translate-z, 0) * 1px));
  background-color: var(--color, black);
}
.back {
  transform: translateX(calc(var(--translate-x, 0) * 1px)) translateY(calc(var(--translate-y, 0) * 1px)) translateZ(calc(var(--translate-z, 0) * 1px)) translateZ(calc(var(--size) * -1px));
  background-color: var(--color, black);
}
</style>
