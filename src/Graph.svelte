
<script>
export class Sgraph {
    constructor(canvas) {
      this.canvas = canvas;
      this.ctx = canvas.getContext("2d");
      this.width = canvas.width;
      this.height = canvas.height;
      this.x = this.width / 2;
      this.y = this.height / 2;
      this.angle = 0;
      this.history = [];
    }
  
    update(accuracy) {
      const maxDeviation = 0.02; // adjust this to control the meter sensitivity
      const deviation = accuracy - 1;
      const deltaAngle = deviation / maxDeviation * Math.PI / 2; // 90 degrees for full-scale deflection
      this.angle += deltaAngle;
  
      // add the current position to the history array
      const position = {
        x: this.x,
        y: this.y
      };
      this.history.push(position);
  
      // trim the history array to keep only the last 100 positions
      if (this.history.length > 100) {
        this.history.shift();
      }
    }
  
    draw() {
      // clear the canvas
      this.ctx.clearRect(0, 0, this.width, this.height);
  
      // draw the history as a line
      this.ctx.beginPath();
      for (let i = 0; i < this.history.length; i++) {
        const position = this.history[i];
        if (i === 0) {
          this.ctx.moveTo(position.x, position.y);
        } else {
          this.ctx.lineTo(position.x, position.y);
        }
      }
      this.ctx.strokeStyle = "gray";
      this.ctx.lineWidth = 2;
      this.ctx.stroke();
  
      // draw the pendulum pin
      const pinLength = 50;
      const pinX = this.x + Math.sin(this.angle) * pinLength;
      const pinY = this.y + Math.cos(this.angle) * pinLength;
      this.ctx.beginPath();
      this.ctx.moveTo(this.x, this.y);
      this.ctx.lineTo(pinX, pinY);
      this.ctx.strokeStyle = "red";
      this.ctx.lineWidth = 5;
      this.ctx.stroke();
      this.ctx.beginPath();
      this.ctx.arc(this.x, this.y, 10, 0, 2 * Math.PI);
      this.ctx.fillStyle = "white";
      this.ctx.fill();
    }
  }
  </script>