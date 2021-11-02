<template>
	<div class="home">
		<div class="title">Trigle Crush</div>
		<div class="score">Score: {{score}}</div>
		<TrigleCurshGame ref="game" @addScore="addScore" @unsolvable="unsolvable"/>
		<div class="btn-again" @click="playAgain">Play Again</div>
	</div>
</template>

<script>
  // @ is an alias to /src
  import TrigleCurshGame from '@/components/TrigleCurshGame.vue'

  export default {
    name: 'Home',
    components: {
      TrigleCurshGame
    },
    data() {
      return {
        score: 0,
      };
    },
    methods: {
      addScore( num ) {
        console.log( 'addScore', num )
        this.score += num;
      },
      unsolvable() {
        alert( 'There is no answer! Please play again!' );
      },
      playAgain() {
        this.score = 0;
        this.$refs.game.playAgain();
      },
    },
  }
</script>

<style lang="scss" scoped>
	.home {
		font-size: 40px;

		.title {
			padding: 50px;
			text-align: center;
			font-weight: bold;
		}

		.score {
			margin-bottom: 10px;
			text-align: center;
		}

		.btn-again {
			margin: 50px auto;
			padding: 10px 20px;
			width: 240px;
			border-radius: 10px;
			text-align: center;
			color: #fff;
			background: #6DDBAA;
		}
	}
</style>