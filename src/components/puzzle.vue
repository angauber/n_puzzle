<template lang="html">
	<div class="theming-container ui container">
		<sui-dropdown
			placeholder="Size"
			selection
			:options="options"
			v-model="current"
			v-on:input="init();"
			style="margin-top: 20px"
		/>
		<sui-container>
			<sui-grid class="ui middle aligned centered" style="margin-top: 10px;">
				<sui-grid-row v-for="column in puzzle">
					<div v-for="nb in column">
						<sui-segment v-if="nb !== 0" v-on:click="play(nb);" style="cursor: pointer; margin-left: 15px; margin-right: 15px; width: 40px; text-align: center;">{{ nb }}</sui-segment>
						<sui-segment v-else style="visibility: hidden; margin-left: 15px; margin-right: 15px; width: 40px">0</sui-segment>
					</div>
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
			solving: false,
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
			puzzle: [],
			final_state: [],
		};
	},
	methods: {
		init: function() {
			let tab = [];
			let k = 1;
			for(let i = 0; i < this.current; i++) {
				tab[i] = [];
				for(let j = 0; j < this.current; j++) {
					tab[i][j] = k;
					k++;
				}
			}
			tab[this.current - 1][this.current - 1] = 0;
			this.final_state = tab;
			this.puzzle = JSON.parse(JSON.stringify(tab));
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
			this.puzzle = tab;
			while (this.is_solvable(this.puzzle) === false) {
				this.shuffle();
			}
		},
		rand_shuffle: function(a) {
    		for (let i = a.length - 1; i > 0; i--) {
        		const j = Math.floor(Math.random() * (i + 1));
        		[a[i], a[j]] = [a[j], a[i]];
    		}
    		return a;
		},
		play: function(nb) {
			let tab = this.puzzle;

			for (let i = 0; i < tab.length; i++) {
				for (let j = 0; j < tab.length; j++) {
					if (tab[i][j] === nb) {
						const res = this.is_movable(tab, i, j)
						if (res) {
							tab[res[0]][res[1]] = tab[i][j];
							tab[i][j] = 0;
							this.puzzle = tab;
							this.$forceUpdate();
						}
						return ;
					}
				}
			}
		},
		is_movable(tab, i, j) {
			if (tab[i][j + 1] === 0) {
				return [i, j+1];
			}
			else if (tab[i][j - 1] === 0) {
				return [i, j-1];
			}
			else if (i > 0 && tab[i - 1][j] === 0) {
				return [i-1, j];
			}
			else if (i < (tab.length - 1) && tab[i + 1][j] === 0) {
				return [i+1, j];
			}
			else {
				return ;
			}
		},
		is_solvable: function(node) {
			let tab = [];
			let inversionCount = 0;
			for (let i = 0; i < this.current; i++) {
				for (let j = 0; j < this.current; j++) {
					tab.push(node[i][j]);
				}
			}
			for (let i = 0; i < tab.length - 1; i++) {
				for (let j = i + 1; j < tab.length; j++) {
					if (tab[i] > tab[j]) {
						inversionCount++;
					}
				}
			}
			if (inversionCount % 2 == 0) {
				return true;
			}
			return false;
		},
		start_solve: function() {
			let t0 = performance.now();

			this.a_star();   // <---- The function you're measuring time for

			let t1 = performance.now();
			console.log("Call to a_star took " + (t1 - t0) + " milliseconds.")
		},
		a_star: function() {
			console.log('starting');
			this.solving = true;
			let current, start, gScore, gScoreIsBest, self;
			let opened = [];
			let closed = [];

			start = JSON.parse(JSON.stringify(this.puzzle));
			start.g = 0;
			start.h = this.h_Hamming(start);
			start.f = start.h;
			start.parent = null;
			opened = [...opened, start];
			while (opened.length !== 0) {
				current = this.get_least_f(opened);
				if (this.is_equal(current, this.final_state)) {
					console.log('finished');
					this.resolve(current);
					return ;
				}
				this.remove_from_array(opened, current);
				closed = [...closed, current];
				self = this;
				this.get_next_nodes(current).forEach(function(neighbor) {
					if (self.is_inside(closed, neighbor) === true) { // does not seem to work.. resultig in infinte loop
						// console.log('already closed');
						return ;
					}
					gScore = current.g + 1;
					gScoreIsBest = false;
					if (self.is_inside(open, neighbor) === false) {               //Elem does not exist in open list
						// console.log('adding in opened');
						gScoreIsBest = true;
						opened = [...opened, neighbor];					 //Adding elem in open list
					}
					else if (gScore < neighbor.g) {
						console.log('lol');							// wer may got a problem here
						gScoreIsBest = true;
					}

					if (gScoreIsBest) {
					  // Found an optimal (so far) path to this node.   Store info on how we got here and
					  //  just how good it really is...
						neighbor.parent = current;
						neighbor.g = gScore;
						neighbor.f = neighbor.g + neighbor.h;
					}
				})
			}
			console.log('no solutions found :( !');
			console.log(current);
		},
		get_next_nodes: function(node) {
			let childs = [];
			let child, pos;
			for (let i = 0; i < this.current; i++) {
				for (let j = 0; j < this.current; j++) {
					pos = this.is_movable(node, i, j);
					if (pos) {
						child = JSON.parse(JSON.stringify(node));
						child[pos[0]][pos[1]] = node[i][j];
						child[i][j] = 0;
						child.g = node.g + 1;
						child.h = this.h_Hamming(child);
						child.f = child.g + child.h;
						childs = [...childs, child];
					}
				}
			}
			return childs;
		},
		get_least_f: function(nodes) {
			let best;
			best = nodes[0];
			nodes.forEach(function(node) {
				if (node.f < best.f) {
					best = node;
				}
			})
			return best;
		},
		h_Hamming: function(tab2) {
			let tab1 = this.final_state;
			let h = 0;
			for(let i = 0; i < this.current; i++) {
				for(let j = 0; j < this.current; j++) {
					if (tab1[i][j] !== tab2[i][j] && tab1[i][j] !== 0) {
						h++;
					}
				}
			}
			return h;
		},
		is_inside: function(haystack, needle) {
			for (let i = 0; i < haystack.length; i++) {
				if (this.is_equal(haystack[i], needle) === true) {
					return true;
				}
			}
			return false;
		},
		is_equal: function(node, tab) {
			for (let i = 0; i < this.current; i++) {
				for (let j = 0; j < this.current; j++) {
					if (node[i][j] !== tab[i][j]) {
						return false;
					}
				}
			}
			return true;
		},
		remove_from_array: function(parent, child) {
			const index = parent.indexOf(child);
			parent.splice(index, 1);
		},
		resolve: function(solved) {
			let solution = [];
			while (solved !== null) {
				solution.unshift(solved);
				solved = solved.parent;
			}
			console.log(solution);
		}
	},
	mounted: function() {
		this.init();
	}
};
</script>
