<template>
	<div class="home">
		<div class="title">Trigle Crush</div>
		<div class="score">Score: {{score}}</div>
		<TrigleCurshGame @addScore="addScore"/>
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
        console.log('addScore', num)
        this.score += num;
      },
    },
  }
</script>

<style lang="scss" scoped>
	.home {
		font-size: 40px;

		.title {
			padding: 20px;
			text-align: center;
			font-weight: bold;
		}

		.score {
			margin-bottom: 10px;
			text-align: center;
		}
	}
</style>