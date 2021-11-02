<template>
	<div class="trigle-crush-game">
		<div class="blocks">
			<template v-for="block in blocks">
				<div class="block" :key="block.id"
				     :class="[block.status, !animating&&((answer[0]==block.x&&answer[1]==block.y)||(answer[2]==block.x&&answer[3]==block.y))?'answer':'']"
				     :x="block.x" :y="block.y" :color="block.color"
				     @touchstart="onTouchStart(block, ...arguments)"
				     @touchmove="onTouchMove(block, ...arguments)">
				</div>
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
        animating: false,
        cells: [],
        blocks: [],
        lastBlockId: 0,
        answer: [],
      };
    },
    mounted() {
      this.init();
    },
    methods: {
      init() {
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
      playAgain() {
        this.init();
      },
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
        if ( this.animating ) return;
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
        if ( !blocks.length ) {
          this.answer = this.__findAnswer();
          if ( !this.answer || !this.answer.length ) {
            this.unsolvable();
          }
          return false;
        }
        this.__crushBlocks( blocks );
        return true;
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
      __crushBlocks( blocks ) {
        this.animating = true;
        Promise.all(
          blocks.map( block => {
            return this.__crushBlock( block ).then( () => {
              this.cells[ block.x ][ block.y ] = {};
              this.blocks.splice( this.blocks.indexOf( block ), 1 );
            } );
          } )
        ).then( () => {
          this.addScore( blocks.length );
          this.__fall( blocks ).then( () => {
            this.animating = false;
            this.__checkCrush();
          } );
        } );
        this.blocks.splice();
      },
      __crushBlock( block ) {
        console.log( '__crushBlock', block.x, block.y )
        block.status = 'crushing';
        return new Promise( resolve => {
          setTimeout( resolve, 300 );
        } );
      },
      __fall( blocks ) {
        this.animating = true;
        let fallingNums = {};
        blocks.sort( ( a, b ) => ( a.y - b.y ) * W + ( a.x - b.x ) ).map( block => {
          if ( !fallingNums[ block.x ] ) fallingNums[ block.x ] = 0;
          return this.__findSubstituteBlock( block, fallingNums[ block.x ]++ );
        } ).forEach( newBlock => {
          newBlock.falling = fallingNums[ newBlock.x ];
        } );
        return Promise.all(
          this.blocks.filter( block => block.falling ).map( block => {
            return this.__moveBlock( block, block.x, block.y - block.falling ).then( () => {
              block.falling = 0;
            } );
          } )
        );
      },
      __findSubstituteBlock( block, fallingNum ) {
        const newBlock = this.generateBlock( block.x, H + fallingNum );
        this.blocks.push( newBlock );

        for ( let y = block.y + 1; y < H; ++y ) {
          if ( this.cells[ block.x ][ y ].id ) {
            this.cells[ block.x ][ y ].falling = this.cells[ block.x ][ y ].falling ? this.cells[ block.x ][ y ].falling + 1 : 1;
          }
        }
        return newBlock;
      },
      __moveBlock( block, x, y ) {
        if ( this.cells[ block.x ] && this.cells[ block.x ][ block.y ] ) this.cells[ block.x ][ block.y ] = {};
        setTimeout( () => {
          this.cells[ x ][ y ] = block;
          block.x = x;
          block.y = y;
        }, 100 );
        return new Promise( resolve => {
          setTimeout( resolve, 300 );
        } );
      },
      __findAnswer() {
        const __checkExchangeBlocks = ( x, y, x2, y2 ) => {
          const __exchangeBlocks = ( x, y, x2, y2 ) => {
            const block = this.cells[ x ][ y ];
            const block2 = this.cells[ x2 ][ y2 ];
            block.x = x2;
            block.y = y2;
            block2.x = x;
            block2.y = y;
            this.cells[ x ][ y ] = block2;
            this.cells[ x2 ][ y2 ] = block;
          };

          __exchangeBlocks( x, y, x2, y2 );
          const result = this.__checkBlockCrush( this.cells[ x ][ y ] ) || this.__checkBlockCrush( this.cells[ x2 ][ y2 ] );
          __exchangeBlocks( x, y, x2, y2 );
          // console.log( x, y, x2, y2, result )
          return result;
        };

        for ( let i = 0; i < this.blocks.length; ++i ) {
          const { x, y } = this.blocks[ i ];
          if ( x < W - 1 ) {
            if ( __checkExchangeBlocks( x, y, x + 1, y ) ) return [ x, y, x + 1, y ];
          }
          if ( y < H - 1 ) {
            if ( __checkExchangeBlocks( x, y, x, y + 1 ) ) return [ x, y, x, y + 1 ];
          }
        }

        return [];
      },
      addScore( num ) {
        this.$emit( 'addScore', num );
      },
      unsolvable() {
        this.$emit( 'unsolvable' );
      },
    },
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
	$X: 7;
	$Y: 7;
	$W: 90px;
	$H: 90px;
	$gap: 10px;

	.trigle-crush-game {
		position: relative;
		height: ($H + $gap) * $Y;

		.blocks {
			position: absolute;
			left: 30px;
			top: 0;

			.block {
				position: absolute;
				width: $W;
				height: $H;
				text-align: center;
				background: #42b983;
				border-radius: 10px;
				transition: all 0.3s;

				&.crushing {
					animation: block-crushing 0.3s ease-in-out both;
					@keyframes block-crushing {
						0% {
							transform: scale(1);
							opacity: 1;
						}
						100% {
							transform: scale(1.5);
							opacity: 0;
						}
					}
				}

				&.answer {
					animation: block-answer 0.5s ease-in-out infinite alternate;
					@keyframes block-answer {
						0% {
							transform: scale(1);
						}
						100% {
							transform: scale(0.9);
						}
					}
				}

				@for $x from 0 to $X {
					@for $y from 0 to $Y * 2 {
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
