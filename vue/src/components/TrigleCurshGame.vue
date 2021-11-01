<template>
	<div class="trigle-crush-game">
		<div class="blocks">
			<template v-for="block in blocks">
				<div class="block" :key="block.id"
				     :x="block.x" :y="block.y" :color="block.color"
				     @touchstart="onTouchStart(block, ...arguments)"
				     @touchmove="onTouchMove(block, ...arguments)"></div>
			</template>
		</div>
	</div>
</template>

<script>
  const W = 7, H = 7;
  export default {
    name: 'TrigleCurshGame',
    props: {},
    data() {
      return {
        cells: [],
        blocks: [],
        lastBlockId: 0,
      };
    },
    mounted() {
      let cells = [];
      let blocks = [];
      for ( let x = 0; x < W; ++x ) {
        cells.push( [] );
        for ( let y = 0; y < H; ++y ) {
          const block = this.generateBlock( x, y );
          cells[ x ].push( block );
          blocks.push( block );
        }
      }
      this.cells = cells;
      this.blocks = blocks;
      this.__checkCrush();
    },
    methods: {
      generateBlock( x, y ) {
        let block = {
          id: ++this.lastBlockId,
          x,
          y,
          color: Math.floor( Math.random() * 7 ),
        };
        return block;
      },
      onTouchStart( block, e ) {
        const touch = e.touches[ 0 ];
        this._touchStartBlockId = block.id;
        this._touchStartBlockX = touch.clientX;
        this._touchStartBlockY = touch.clientY;
      },
      onTouchMove( block, e ) {
        if ( this.animating ) return;
        if ( this._touchStartBlockId !== block.id ) return;
        const touch = e.touches[ 0 ];
        const deltaX = touch.clientX - this._touchStartBlockX;
        const deltaY = touch.clientY - this._touchStartBlockY;
        const threshold = 30;
        if ( Math.abs( deltaX ) > threshold && Math.abs( deltaY ) < threshold / 2 ) {
          if ( deltaX < 0 ) {
            this.exchangeBlock( block, 'left' );
          } else {
            this.exchangeBlock( block, 'right' );
          }
          this._touchStartBlockId = null;
        } else if ( Math.abs( deltaX ) < threshold / 2 && Math.abs( deltaY ) > threshold ) {
          if ( deltaY < 0 ) {
            this.exchangeBlock( block, 'up' );
          } else {
            this.exchangeBlock( block, 'down' );
          }
          this._touchStartBlockId = null;
        } else if ( Math.abs( deltaX ) > threshold / 2 && Math.abs( deltaY ) > threshold / 2 ) {
          this._touchStartBlockId = null;
        }
      },
      exchangeBlock( block, direction ) {
        const { x, y } = block;
        let x2, y2;
        switch ( direction ) {
          case 'left':
            x2 = x - 1;
            y2 = y;
            break;
          case 'right':
            x2 = x + 1;
            y2 = y;
            break;
          case 'up':
            x2 = x;
            y2 = y + 1;
            break;
          case 'down':
            x2 = x;
            y2 = y - 1;
            break;
        }
        if ( x2 < 0 || x2 >= W || y2 < 0 || y2 >= H ) return;
        console.log( x, y, direction, x2, y2 )
        this.__exchageBlocks( x, y, x2, y2 ).then( () => {
          if ( !this.__checkCrush() ) {
            this.__exchageBlocks( x, y, x2, y2 );
          }
        } );
      },
      __exchageBlocks( x, y, x2, y2 ) {
        const block = this.cells[ x ][ y ];
        const block2 = this.cells[ x2 ][ y2 ];
        block.x = x2;
        block.y = y2;
        block2.x = x;
        block2.y = y;
        this.cells[ x ][ y ] = block2;
        this.cells[ x2 ][ y2 ] = block;
        this.animating = true;

        return new Promise( resolve => {
          setTimeout( () => {
            this.animating = false;
            resolve();
          }, 300 );
        } );
      },
      __checkCrush() {
        let blocks = [];
        this.blocks.forEach( block => {
          if ( blocks.indexOf( block ) < 0 ) {
            if ( this.__checkBlockCrush( block ) ) blocks.push( block );
          }
        } );
        if ( blocks.length ) {
          this.__crush( blocks );
          return true;
        }
        return false;
      },
      __checkBlockCrush( block ) {
        const { x, y, color } = block;
        if ( x > 1 && this.cells[ x - 2 ][ y ].color === color && this.cells[ x - 1 ][ y ].color === color ) return true;
        if ( x > 0 && x < W - 1 && this.cells[ x - 1 ][ y ].color === color && this.cells[ x + 1 ][ y ].color === color ) return true;
        if ( x < W - 2 && this.cells[ x + 1 ][ y ].color === color && this.cells[ x + 2 ][ y ].color === color ) return true;
        if ( y > 1 && this.cells[ x ][ y - 2 ].color === color && this.cells[ x ][ y - 1 ].color === color ) return true;
        if ( y > 0 && y < H - 1 && this.cells[ x ][ y - 1 ].color === color && this.cells[ x ][ y + 1 ].color === color ) return true;
        if ( y < H - 2 && this.cells[ x ][ y + 1 ].color === color && this.cells[ x ][ y + 2 ].color === color ) return true;
        return false;
      },
      __crush( blocks ) {
        blocks.forEach( block => {
          this.cells[ block.x ][ block.y ] = {};
          this.blocks.splice( this.blocks.indexOf( block ), 1 );
        } );
        this.addScore( blocks.length );
      },
      addScore( num ) {
        this.$emit( 'addScore', num );
      },
    },
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
	.trigle-crush-game {
		position: relative;

		.blocks {
			position: absolute;
			left: 30px;
			top: 0;

			.block {
				$X: 7;
				$Y: 7;
				$W: 90px;
				$H: 90px;
				$gap: 10px;

				position: absolute;
				width: $W;
				height: $H;
				text-align: center;
				background: #42b983;
				border-radius: 10px;
				transition: all 0.3s;

				@for $x from 0 to $X {
					@for $y from 0 to $Y {
						&[x="#{$x}"][y="#{$y}"] {
							left: $x * ($W + $gap);
							top: ($Y - $y - 1) * ($H + $gap);
						}
					}
				}

				&[color="0"] {
					background: #F67777;
				}

				&[color="1"] {
					background: #116611;
				}

				&[color="2"] {
					background: #4F60AF;
				}

				&[color="3"] {
					background: #AA55A5;
				}

				&[color="4"] {
					background: #D4C23A;
				}

				&[color="5"] {
					background: #6DDBAA;
				}

				&[color="6"] {
					background: #ADABAA;
				}
			}
		}
	}
</style>
