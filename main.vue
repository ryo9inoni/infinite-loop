/**
 * HTMLでの呼び出し
 *   
 *  top
 */


<script>
  import CONFIG from '../modules/_config';

  export default {
    props: {
    },
    created () {        
    },
    data: function () {
      return {
        works: [],
        select: [],
        random: {
          data: [],
          index: 0
        },
        pos: {
          x: 0,
          y: 0,
          before: {
            x: 0,
            y: 0,
          },
          after:{
            x: 0,
            y: 0,
          }
        },
        flag: {
          on: false,
          move: false
        },
        row: {
          h: 175,
          comput: Math.ceil(window.innerHeight/175),
        },
        col: {
          w: 95,
          comput: Math.ceil(window.innerWidth/95),
        },
        stage: {},
        continer: {},
        el_balloon: {
          top: [0, 50],
          left: [0, 10],
          margin: [16, 10],
          padding: [40, 32],
          w: 55,
          h: 75,
          radius: [8, 16],
          color: '#0A460F',
          triangle: {
            x: [20, 52],
            y: [74, 20],
            m: ['_pc', '_sp']
          }
        }, 
        el_title: {
          x: [20, 8],
          y: [14, 16],
          font: {
            base: ['48px TBGoStdB-C6', '40px TBGoStdB-C6'],
            ie: ['bold 48px sans-serif', 'bold 40px sans-serif'],
          },
          color: '#ffffff'
        },
        el_no: {
          x: [60, 67],
          y: [86, 42],
          text: 'No.',
          font: ['12px TTCommons-Medium', '11px TTCommons-Medium'],
          color: '#0A460F',
        },
        el_id: {
          x: [0, 70],
          y: [82, 0],
          font: ['17px TTCommons-Medium', '16px TTCommons-Medium'],
          color: '#0A460F',
        },
        el_slash: {
          x: [0, 70],
          y: [82, 0],
          text: '/',
          font: ['14px TTCommons-Medium', '14px TTCommons-Medium'],
          color: '#0A460F',
        },
        el_course: {
          x: [0, 70],
          y: [82, 0],
          font: {
            base: ['13px TBGoStdB-C6', '12px TBGoStdB-C6'],
            ie: ['bold 13px sans-serif', 'bold 12px sans-serif'],
          },
          color: '#0A460F'
        },
      };
    },
    mounted () {
      const canvas = document.querySelector('#canvas');
      const handleTick = () => {
        if(!this.flag.on) {
          if(this.mode === 0){
            this.container.x -= 1;
          } else if(this.mode === 1){
            this.container.y += 1;
          }
        }
        this.Update();
        this.stage.update();
      }
      const deviceScreen = () => {
        if (window.devicePixelRatio) {
          canvas.width *= devicePixelRatio;
          canvas.height *= devicePixelRatio;
          canvas.style.width = String(canvas.width / devicePixelRatio) + "px";
          canvas.style.height = String(canvas.height / devicePixelRatio) + "px";
          this.stage.scaleX = this.stage.scaleY = window.devicePixelRatio;
        }
      }

      this.stage = new createjs.Stage('canvas');
      this.stage.canvas.width = window.innerWidth;
      this.stage.canvas.height = window.innerHeight;  
      this.container = new createjs.Container();
      this.stage.addChild(this.container);
      this.stage.enableMouseOver();
      
      if(this.browser === 'ie'){
          window.addEventListener('load', () => {
            const topScreen = document.querySelector('.top__screen');
            this.ie_On(topScreen);
          });
      }else{
        createjs.Touch.enable(this.stage);
      }
      
      deviceScreen();
      window.addEventListener('resize', () => {
        this.stage.canvas.width = window.innerWidth;
        this.stage.canvas.height = window.innerHeight;
        deviceScreen();
      });
      
      createjs.Ticker.timingMode = createjs.Ticker.RAF;
      createjs.Ticker.addEventListener("tick", handleTick);
    },
    computed: {
      getWorks () {
        return this.$store.state.works;
      },
      browser () {
        return CONFIG.IS_IE ? 'ie' : 'base';
      },
      mode () {
        return window.innerWidth >= 768 ? 0 : 1;
      },
    },
    watch: {
      getWorks (data) {
        if(data.length > 0) {
          this.works = data;

          // 乱数生成
          const min = 0; 
          const max = data.length-1;
          for(let i = min; i <= max; i++){
            while(true){
              const tmp = intRandom(min, max);
              if(!this.random.data.includes(tmp)){
                this.random.data.push(tmp);
                break;
              }
            }
          }
          function intRandom(min, max){
            return Math.floor(Math.random() * (max - min + 1)) + min;
          }

          // 初期描画
          this.Draw();
        }
      }
    },
    methods: {
      Draw () {
        const count = this.mode === 0 ? this.row.comput * 2 : this.col.comput * 2;
        for (let i = 0; i < count; i++) {
          const block = new createjs.Container();
          block.name = 'block';
          this.mode === 0 ? block.y = this.row.h * i : block.x = this.col.w * i;
          this.container.addChild(block);
        }
        this.container.x = -this.stage.canvas.width / 1.5;
        this.container.y = -this.stage.canvas.height / 1.5;
      },     
      Click (event) {
        if(this.mode === 0) this.flag.move = false;
      },
      On (event) {
        this.flag.on = true;
        this.pos.x = this.stage.mouseX - this.container.x;
        this.pos.y = this.stage.mouseY - this.container.y;
      },
      Move (on) {
        if(on){
          const afterPosX = this.stage.mouseX - this.container.x;
          const posDiff = Math.sign(afterPosX - this.pos.x) === -1 ? (afterPosX - this.pos.x) * -1 : afterPosX - this.pos.x;
          if(posDiff > 3) this.flag.move = true;

          const xNum = (this.pos.x - this.stage.mouseX) * -1;
          const yNum = (this.pos.y - this.stage.mouseY) * -1;
          createjs.Tween.get(this.container).to({x: xNum}, 600, createjs.Ease.cubicOut);
          createjs.Tween.get(this.container).to({y: yNum}, 600, createjs.Ease.cubicOut);
        }
      },
      ie_On (topScreen){
        topScreen.addEventListener('mousedown', () => {
          this.flag.on = true;
          this.pos.x = this.stage.mouseX - this.container.x;
          this.pos.y = this.stage.mouseY - this.container.y;
        });
      },
      Out (event) {
        this.flag.on = false;
        if(this.mode === 1) this.flag.move = false;
      },
      /**
       * 吹き出しを生成する
       * @param {Number} x 追加位置
       * @param {Number} y 追加位置
       * @param {Boolean} before 上下左右
       */
      CreateBalloon (x = 0, y = 0, before = false, edit = null) {

        while(this.works[this.random.data[this.random.index]].title === "") {
          this.random.index++;
          if(this.random.index >= this.random.data.length){
            this.random.index = 0;
          }
        }

        // 吹き出しのタイトル
        let text;
        if(this.mode === 0){
          text = this.works[this.random.data[this.random.index]].title;
        }else if(this.mode === 1){
          let replace = this.works[this.random.data[this.random.index]].title
            .replace('ー', '︲')
            .replace('「', '﹁')
            .replace('」', '﹂')
            .replace('『', '﹁')
            .replace('』', '﹂')
            .replace('\"', '');
          text = replace.split('').join('\n');
        }

        const titleEl = edit === null ? new createjs.Text(text, this.el_title.font[this.browser][this.mode], this.el_title.color) : edit.getChildByName('title');
        titleEl.name = 'title';
        titleEl.x = this.el_title.x[this.mode];
        titleEl.y = this.el_title.y[this.mode];
        titleEl.lineHeight = this.mode === 0 ? 0 : 40;
        edit === null ? null : titleEl.text = text;

        // 吹き出しのNo.
        const noEl = edit === null ? new createjs.Text(this.el_no.text, this.el_no.font[this.mode], this.el_no.color) : edit.getChildByName('no');
        noEl.name = 'no';
        noEl.x = this.el_no.x[this.mode];
        noEl.y = this.el_no.y[this.mode];
        noEl.rotation = this.mode === 0 ? 0 : 90;

        // 吹き出しのid
        const idEl = edit === null ? new createjs.Text(this.works[this.random.data[this.random.index]].id, this.el_id.font[this.mode], this.el_id.color) : edit.getChildByName('id');
        idEl.name = 'id';
        idEl.x = this.mode === 0 ? this.el_no.x[this.mode] + noEl.getMeasuredWidth() + 1 : this.el_id.x[this.mode];
        idEl.y = this.mode === 0 ? this.el_id.y[this.mode] : this.el_no.y[this.mode] + noEl.getMeasuredWidth() + 1;
        idEl.rotation = this.mode === 0 ? 0 : 90;
        edit === null ? null : idEl.text = this.works[this.random.data[this.random.index]].id;

        const slashEl = edit === null ? new createjs.Text(this.el_slash.text, this.el_slash.font[this.mode], this.el_slash.color) : edit.getChildByName('slash');
        slashEl.name = 'slash';
        slashEl.x = this.mode === 0 ? idEl.x + idEl.getMeasuredWidth() + 4 : this.el_slash.x[this.mode];
        slashEl.y = this.mode === 0 ? this.el_slash.y[this.mode] : idEl.y + idEl.getMeasuredWidth() + 4;
        slashEl.rotation = this.mode === 0 ? 0 : 90;

        const courseEl = edit === null ? new createjs.Text(this.works[this.random.data[this.random.index]].course_name, this.el_course.font[this.browser][this.mode], this.el_course.color) : edit.getChildByName('course');
        courseEl.name = 'course';
        courseEl.x = this.mode === 0 ? slashEl.x + slashEl.getMeasuredWidth() + 4 : this.el_course.x[this.mode];
        courseEl.y = this.mode === 0 ? this.el_course.y[this.mode] : slashEl.y + slashEl.getMeasuredWidth() + 4;
        courseEl.rotation = this.mode === 0 ? 0 : 90;
        edit === null ? null : courseEl.text = this.works[this.random.data[this.random.index]].course_name;

        titleEl.w = this.mode === 0 ? titleEl.getMeasuredWidth() + this.el_balloon.padding[this.mode] : titleEl.getMeasuredHeight() + this.el_balloon.padding[this.mode];
        courseEl.w = this.mode === 0 ? courseEl.getMeasuredWidth() + courseEl.x : courseEl.getMeasuredWidth() + courseEl.y;
        const elW = titleEl.w > courseEl.w ? titleEl.w : courseEl.w;
        
        // 吹き出しの形
        const shapeEl = edit === null ? new createjs.Shape() : edit.getChildByName('shape'); shapeEl.graphics.clear();
        shapeEl.name = 'shape';
        shapeEl.graphics.beginFill(this.el_balloon.color);
        shapeEl.graphics.drawRoundRect(
          0,
          0,
          this.mode === 0 ? titleEl.w : this.el_balloon.w,
          this.mode === 0 ? this.el_balloon.h : titleEl.w,
          this.el_balloon.radius[this.mode],
          this.el_balloon.radius[this.mode]
        );
        
        // 吹き出しのとんがり
        const triangleEl = edit === null ? new createjs.Bitmap('/assets/images/triangle' + this.el_balloon.triangle.m[this.mode] + '.png') : edit.getChildByName('triangle');
        triangleEl.name = 'triangle';
        triangleEl.x = this.el_balloon.triangle.x[this.mode];
        triangleEl.y = this.el_balloon.triangle.y[this.mode];

        // 吹き出しの要素
        const balloonEl = edit === null ? new createjs.Container() : edit;
        balloonEl.width = elW;

        balloonEl.addChild(shapeEl);
        balloonEl.addChild(triangleEl);
        balloonEl.addChild(titleEl);
        balloonEl.addChild(noEl);
        balloonEl.addChild(idEl);
        balloonEl.addChild(slashEl);
        balloonEl.addChild(courseEl);

        balloonEl.name = 'balloon';
        balloonEl.alpha = 0;

        if(this.mode === 0){
          if(before) {
            // 左に追加
            balloonEl.x = x - this.el_balloon.margin[this.mode] - elW;
          } else {
            // 右に追加
            balloonEl.x = x + this.el_balloon.margin[this.mode];
          }
          balloonEl.y = this.el_balloon.top[y];
        } else if(this.mode === 1){
          if(before) {
            // 上に追加
            balloonEl.y = y - this.el_balloon.margin[this.mode] - elW;
          } else {
            // 下に追加
            balloonEl.y = y + this.el_balloon.margin[this.mode];
          }
          balloonEl.x = this.el_balloon.left[x];
        }

        balloonEl.data = this.works[this.random.data[this.random.index]];
        balloonEl.addEventListener('mouseover', handleOver.bind(balloonEl));
        balloonEl.addEventListener('mouseout', handleOut.bind(balloonEl));
        balloonEl.addEventListener('click', handleClick.bind(balloonEl));
        const screen = document.querySelector('.screen');
        function handleOver() {
          screen.style.cursor = 'pointer';
          createjs.Tween.get(this).to({alpha:0.8}, 600, createjs.Ease.cubicOut);
        }
        function handleOut() {
          screen.style.cursor = 'auto';
          createjs.Tween.get(this).to({alpha:1}, 600, createjs.Ease.cubicOut);
        }
        const self = this;
        const body = document.querySelector('body');
        const detail = document.querySelector('.detail');
        const detail_id = detail.querySelector('.detail__id');
        const detail_course = detail.querySelector('.detail__course');
        const detail_description = detail.querySelector('.detail__description');
        const detail_thumbnail = detail.querySelector('.detail__thumbnail');
        function handleClick() {
          if(!self.flag.move){
            detail_id.innerText = this.data.id;
            detail_course.innerText = this.data.course_name;
            let textEdit = ''; 
            for (let i = 0; i < this.data.description.length; i++) {
              if(this.data.description[i] === "。"){
                textEdit+= '<span class=-editFont>' + this.data.description[i] + '</span>';
              }else{
                textEdit+= this.data.description[i];
              } 
            }
            detail_description.innerHTML = textEdit;
            detail_thumbnail.src = '';
            switch (this.data.thumbnail.length) {
              case 1: this.data.thumbnail = '000'+this.data.thumbnail; break;
              case 2: this.data.thumbnail = '00'+this.data.thumbnail; break;
              case 3: this.data.thumbnail = '0'+this.data.thumbnail; break;
            }
            detail_thumbnail.src = '/assets/images/detail/'+this.data.thumbnail+'.jpg';
            detail_thumbnail.onerror = () => {
              detail_thumbnail.src = '/assets/images/detail/no-image.png';
            } 
            detail.classList.add('-active');
            body.style.overflow = 'hidden';
          }
        }
        
        this.random.index++;
        if(this.random.index >= this.random.data.length){
          this.random.index = 0;
        }

        if(edit === null) return balloonEl;
      },
      Update () {
        if(!this.container.children || this.container.children.length === 0) {
          return;
        }

        // ドラッグmove
        this.Move(this.flag.on);
        
        if(this.mode === 0){
          if(this.container.children[0].y + this.container.y > this.stage.canvas.height * -0.2) {
            // 上に空白がある場合、一番下のblockを一番上に移動
            const fistChild = this.container.getChildAt(0);
            const lastChild = this.container.getChildAt(this.container.children.length-1);
            let balloon_x = this.container.x * -1;
            let balloon_y = 0;
            lastChild.children.forEach(balloon => {
              this.CreateBalloon(balloon_x, balloon_y, false, balloon);
              balloon_x += balloon.width + this.el_balloon.margin[this.mode];
              balloon_y = (balloon_y === 0)? 1 : 0;
              createjs.Tween.get(balloon).to({alpha:1}, 600, createjs.Ease.cubicOut);
            });
            lastChild.y = fistChild.y - this.row.h;
            this.container.addChildAt(lastChild, 0);
          } else if (this.container.children[this.container.children.length-1].y + this.container.y + this.row.h < this.stage.canvas.height * 1.2) {
            const fistChild = this.container.getChildAt(0);
            const lastChild = this.container.getChildAt(this.container.children.length-1);
            let balloon_x = this.container.x * -1;
            let balloon_y = 0;
            lastChild.children.forEach(balloon => {
              this.CreateBalloon(balloon_x, balloon_y, false, balloon);
              balloon_x += balloon.width + this.el_balloon.margin[this.mode];
              balloon_y = (balloon_y === 0)? 1 : 0;
              createjs.Tween.get(balloon).to({alpha:1}, 600, createjs.Ease.cubicOut);
            });
            fistChild.y = lastChild.y + this.row.h;
            this.container.addChildAt(fistChild, this.container.children.length-1);
          }
        }else if(this.mode === 1){
          if(this.container.children[0].x + this.container.x > this.stage.canvas.width * -0.1) {
            const fistChild = this.container.getChildAt(0);
            const lastChild = this.container.getChildAt(this.container.children.length-1);
            let balloon_x = 0;
            let balloon_y = this.container.y * -1;
            lastChild.children.forEach(balloon => {
              this.CreateBalloon(balloon_x, balloon_y, false, balloon);
              balloon_x = (balloon_x === 0) ? 1 : 0;
              balloon_y += balloon.width + this.el_balloon.margin[this.mode];
              createjs.Tween.get(balloon).to({alpha:1}, 600, createjs.Ease.cubicOut);
            });
            lastChild.x = fistChild.x - this.col.w;
            this.container.addChildAt(lastChild, 0);  
          } else if (this.container.children[this.container.children.length-1].x + this.container.x + this.col.w < this.stage.canvas.width * 1.1) {
            const fistChild = this.container.getChildAt(0);
            const lastChild = this.container.getChildAt(this.container.children.length-1);
            let balloon_x = 0;
            let balloon_y = this.container.y * -1;
            lastChild.children.forEach(balloon => {
              this.CreateBalloon(balloon_x, balloon_y, false, balloon);
              balloon_x = (balloon_x === 0) ? 1 : 0;
              balloon_y += balloon.width + this.el_balloon.margin[this.mode];
              createjs.Tween.get(balloon).to({alpha:1}, 600, createjs.Ease.cubicOut);
            });
            fistChild.x = lastChild.x + this.col.w;
            this.container.addChildAt(fistChild, this.container.children.length-1);
          }
        }
        
        this.container.children.forEach((block, index) => { 
          if(block.children.length === 0) {
            // 吹き出しが０個の場合
            const newBalloon = this.mode === 0 ? this.CreateBalloon(this.container.x, 0) : this.CreateBalloon(0, this.container.y);
            this.container.children[index].addChild(newBalloon);
            newBalloon.alpha = 1;
            
            if(this.mode === 0){
              // 吹き出し追加
              while (block.children[block.children.length-1].x < this.stage.canvas.width) {
                // 先頭の吹き出しと高さをずらす
                const y = block.children[block.children.length-1].y === this.el_balloon.top[0] ? 1 : 0;
                const newBalloon = this.CreateBalloon(block.children[block.children.length-1].x + block.children[block.children.length-1].width, y, false);
                this.container.children[index].addChild(newBalloon);
                newBalloon.alpha = 1;
              }
            } else if(this.mode === 1){
              // 吹き出し追加
              while (block.children[block.children.length-1].y < this.stage.canvas.height) {
                // 先頭の吹き出しと横をずらす
                const x = block.children[block.children.length-1].x === this.el_balloon.left[0] ? 1 : 0;
                const newBalloon = this.CreateBalloon(x, block.children[block.children.length-1].y + block.children[block.children.length-1].width, false);
                this.container.children[index].addChild(newBalloon);
                newBalloon.alpha = 1;
              }
            }
          }else{
            if(this.mode === 0){
              if(block.children[0].x + this.container.x > 0) {
                // 左に空白

                // 先頭の吹き出しと高さをずらす
                let firstChild = this.container.children[index].getChildAt(0);
                let lastChild = this.container.children[index].getChildAt(block.children.length-1);
                let y = firstChild.y === this.el_balloon.top[0] ? this.el_balloon.top[1] : this.el_balloon.top[0];
                this.CreateBalloon(0, 0, true, lastChild);
                lastChild.x = firstChild.x - lastChild.width - this.el_balloon.margin[this.mode];
                lastChild.y = y;
                this.container.children[index].addChildAt(lastChild, 0);
                createjs.Tween.get(lastChild).to({alpha:1}, 600, createjs.Ease.cubicOut);
              } else if (block.children[block.children.length-1].x + block.children[block.children.length-1].width + this.container.x < this.stage.canvas.width) {
                // 右に空白

                // 先頭の吹き出しと高さをずらす
                const firstChild = this.container.children[index].getChildAt(0);
                const lastChild = this.container.children[index].getChildAt(block.children.length-1);
                const y = lastChild.y === this.el_balloon.top[0] ? this.el_balloon.top[1] : this.el_balloon.top[0];
                this.CreateBalloon(0, 0, false, firstChild);
                firstChild.x = lastChild.x + lastChild.width + this.el_balloon.margin[this.mode];
                firstChild.y = y;
                this.container.children[index].addChildAt(firstChild, block.children.length-1);
                createjs.Tween.get(firstChild).to({alpha:1}, 600, createjs.Ease.cubicOut);
              }
            } else if(this.mode === 1){
              if(block.children[0].y + this.container.y > 0) {
                // 上に空白

                // 先頭の吹き出しと高さをずらす
                const firstChild = this.container.children[index].getChildAt(0);
                const lastChild = this.container.children[index].getChildAt(block.children.length-1);
                const x = firstChild.x === this.el_balloon.left[0] ? this.el_balloon.left[1] : this.el_balloon.left[0];
                this.CreateBalloon(0, 0, true, lastChild);
                lastChild.y = firstChild.y - lastChild.width - this.el_balloon.margin[this.mode];
                lastChild.x = x;
                this.container.children[index].addChildAt(lastChild, 0);
                createjs.Tween.get(lastChild).to({alpha:1}, 600, createjs.Ease.cubicOut);
              } else if (block.children[block.children.length-1].y + block.children[block.children.length-1].width + this.container.y < this.stage.canvas.height) {
                // 下に空白

                // 先頭の吹き出しと高さをずらす
                const firstChild = this.container.children[index].getChildAt(0);
                const lastChild = this.container.children[index].getChildAt(block.children.length-1);
                const x = lastChild.x === this.el_balloon.left[0] ? this.el_balloon.left[1] : this.el_balloon.left[0];
                this.CreateBalloon(0, 0, false, firstChild);
                firstChild.y = lastChild.y + lastChild.width + this.el_balloon.margin[this.mode];
                firstChild.x = x;
                this.container.children[index].addChildAt(firstChild, block.children.length-1);
                createjs.Tween.get(firstChild).to({alpha:1}, 600, createjs.Ease.cubicOut);
              }
            }
          }
        });
      }
    }
  };
</script>

<template>
  <div class="top" @mousedown="On" @touchstart="On" @touchend="Out" @touchout.capture="Out" @mouseleave="Out" @mouseup.capture="Out" @click="Click">
    <div class="screen">
      <canvas id="canvas"></canvas>
    </div>
  </div>
</template>