<template lang="html">
	<div class="theming-container ui container" style="border: 1px solid grey;">
		<sui-dropdown
			placeholder="Size"
			selection
			:options="options"
			v-model="current"
			v-on:input="init();"
			style="margin-top: 20px"
		/>
		<sui-container>
			<sui-grid>
				<sui-grid-row>
					<sui-grid-column style="margin-left: 35%;">
					</sui-grid-column>
					<sui-grid-column v-for="column in columns" style="margin-top: 30px;">
						<div v-for="nb in column">
							<sui-segment v-if="nb !== 0" v-on:click="play(nb);" style="cursor: pointer; margin-top: 10px">{{ nb }}</sui-segment>
							<sui-segment v-else style="visibility: hidden; margin-top: 10px">0</sui-segment>
						</div>
					</sui-grid-column>
				</sui-grid-row>
			</sui-grid>
		</sui-container>
		<sui-button v-on:click="shuffle()" basic color="orange" style="margin: 50px 20px">Shuffle</sui-button>
		<sui-button v-on:click="start_solve()" basic color="yellow">Solve</sui-button>
	</div>
</template>

<script>
export default {
	name: 'puzzle',
	data() {
		return {
			current: 3,
			options: [{
				text: '3 x 3',
				value: 3,
			}, {
				text: '4 x 4',
				value: 4,
			}, {
				text: '5 x 5',
				value: 5,
			},
			{
				text: '6 x 6',
				value: 6,
			}],
			columns: []
		};
	},
	methods: {
		init: function() {
			let tab = [];
			let k = 0;
			for(let i = 0; i < this.current; i++) {
				tab[i] = [];
				for(let j = 0; j < this.current; j++) {
					tab[i][j] = k;
					k++;
				}
			}
			this.columns = this.rand_shuffle(tab);
		},
		shuffle: function() {
			const length = this.current * this.current;
			let shuffle = [];
			let tab = [];
			let k = 0;
			for (let i = 0; i < length; i++) {
				shuffle.push(i);
			}
			shuffle = this.rand_shuffle(shuffle);
			for(let i = 0; i < this.current; i++) {
				tab[i] = [];
				for(let j = 0; j < this.current; j++) {
					tab[i][j] = shuffle[k];
					k++;
				}
			}
			this.columns = tab;
		},
		rand_shuffle: function(a) {
    		for (let i = a.length - 1; i > 0; i--) {
        		const j = Math.floor(Math.random() * (i + 1));
        		[a[i], a[j]] = [a[j], a[i]];
    		}
    		return a;
		},
		play: function(nb) {
			const tab = this.columns;

			for (let i = 0; i < tab.length; i++) {
				for (let j = 0; j < tab.length; j++) {
					if (tab[i][j] === nb) {
						if (tab[i][j + 1] === 0) {
							tab[i][j + 1] = tab[i][j];
							tab[i][j] = 0;
							this.$forceUpdate();
							return ;
						}
						else if (tab[i][j - 1] === 0) {
							tab[i][j - 1] = tab[i][j];
							tab[i][j] = 0;
							this.$forceUpdate();
							return ;
						}
						else if (i > 0 && tab[i - 1][j] === 0) {
							tab[i - 1][j] = tab[i][j];
							tab[i][j] = 0;
							this.$forceUpdate();
							return ;
						}
						else if (i < (tab.length - 1) && tab[i + 1][j] === 0) {
							tab[i + 1][j] = tab[i][j];
							tab[i][j] = 0;
							this.$forceUpdate();
							return ;
						}
					}
				}
			}
		},
	},
	mounted: function() {
		this.init();
	}
};
</script>
