<template>
	<div>
		<div class="d-flex">
			<table class="board">
				<tr v-for="(tr, y) in boardCells" :key="y">
					<td v-for="(cell, x) in tr" :key="x" @click="clicked(y, x)">
						<span v-if="cell === 1" class="black"></span>
						<span v-if="cell === -1" class="white"></span>
						<span v-if="cell === 0"></span>
					</td>
				</tr>
			</table>
			<div>
				<div class="stone d-flex align-center">
					<span class="black"></span>
					<div>{{ getStonesCount[0] }}</div>
				</div>
				<div class="stone d-flex align-cente">
					<span class="white"></span>
					<div>{{ getStonesCount[1] }}</div>
				</div>
				<div class="d-flex align-cente">
					<span class="blank"></span>
					<div>{{ getStonesCount[2] }}</div>
				</div>
			</div>
		</div>
		<div class="d-flex">
			<div>
				<div class="msg">{{ message }}</div>

				<div>取得石数テーブル</div>
				<table class="ai-board">
					<tr v-for="(tr, y) in canTakeStones" :key="y">
						<td v-for="(count, x) in tr" :key="x">
							<span v-if="count == 0">{{ count }}</span>
							<span
								v-else-if="count >= 1"
								class="pointer"
								@click="showReverce(y, x)"
								>{{ count }}</span
							>
							<span v-else>{{ count }}</span>
						</td>
					</tr>
				</table>
			</div>
			<div class="info">
				<div>取得可能な石座標</div>
				<div>座標:(x,y)</div>
				<div v-for="(flag, i) in showArray" :key="i">
					（ {{ flag[1] + 1 }} , {{ flag[0] + 1 }} ）
				</div>
			</div>
		</div>
	</div>
</template>

<script>
export default {
	data() {
		return {
			turn: 1,
			playerTrun: 1,
			showArray: [],
			message: "あなたは先行(黒)です",
			boardCells: [
				[0, 0, 0, 0, 0, 0],
				[0, 0, 0, 0, 0, 0],
				[0, 0, -1, 1, 0, 0],
				[0, 0, 1, -1, 0, 0],
				[0, 0, 0, 0, 0, 0],
				[0, 0, 0, 0, 0, 0],
			],
			// boardCells: [
			// 	[-1, -1, -1, 1, 1, 1],
			// 	[-1, -1, -1, 1, 1, 1],
			// 	[-1, -1, -1, 1, 1, 1],
			// 	[1, 1, 1, -1, -1, 0],
			// 	[1, 1, 1, -1, -1, -1],
			// 	[1, 1, 1, -1, -1, -1],
			// ],
			canTakeStones: [[], [], [], [], [], []],
			reverceFlagArray: [[], [], [], [], [], []],
			direction: [
				[-1, 0], //上
				[-1, 1], //右上
				[0, 1], //右
				[1, 1], //右下
				[1, 0], //下
				[1, -1], //左下
				[0, -1], //左
				[-1, -1], //左上
			],
		};
	},
	methods: {
		clicked(y, x) {
			if (this.canPutStone(y, x)) {
				this.boardCells[y].splice(x, 1, this.turn);
				this.reverceStone(y, x);
				this.changeTurn();
				this.updateStoneStates();
				setTimeout(this.AiTurnMove, 1000);
			} else {
				this.message = "置けないよ";
			}
		},
		updateStoneStates() {
			this.boardCells.forEach((tr, y) => {
				tr.forEach((cell, x) => {
					if ((cell === 1) | (cell === -1)) {
						this.canTakeStones[y].splice(x, 1, "s");
						this.reverceFlagArray[y].splice(x, 1, []);
					} else {
						this.checkDirection(y, x);
					}
				});
			});
		},
		checkDirection(y, x) {
			let totalReverceStone = 0;
			let childFlagPointArray = [];
			// 各方向へ走査
			this.direction.forEach((dir) => {
				let hasMyStone = false;
				let reverceStone = 0;
				let directionFlags = [];
				let current_y = y + dir[0];
				let current_x = x + dir[1];
				while (this.checkEnd(current_y, current_x)) {
					// 相手の石を発見する
					if (this.boardCells[current_y][current_x] === 0 - this.turn) {
						reverceStone++;
						directionFlags.push([current_y, current_x]);
						current_y += dir[0];
						current_x += dir[1];
						continue;
						// 自分の石を発見する
					} else if (this.boardCells[current_y][current_x] === this.turn) {
						hasMyStone = true;
						break;
						// 空欄の場合
					} else {
						hasMyStone = false;
						break;
					}
				}
				if (hasMyStone & (reverceStone >= 1)) {
					totalReverceStone += reverceStone;
					directionFlags.forEach((flag) => {
						childFlagPointArray.push(flag);
					});
				}
			});
			// console.log("y:" + y + " x:" + x + "トータル" + totalReverceStone);
			this.canTakeStones[y].splice(x, 1, totalReverceStone);
			this.reverceFlagArray[y].splice(x, 1, childFlagPointArray);
		},
		checkEnd(y, x) {
			// 走査方向に対し、5 >= y >= 0 かつ 5 >= x >= 0の範囲内ならtrue
			return (5 >= y) & (y >= 0) & (5 >= x) & (x >= 0);
		},
		canPutStone(y, x) {
			if ((this.canTakeStones[y][x] == 0) | (this.canTakeStones[y][x] == "s")) {
				return false;
			} else {
				return true;
			}
		},
		reverceStone(y, x) {
			let childFlagPoints = this.reverceFlagArray[y][x];
			childFlagPoints.forEach((flagPoint) => {
				this.boardCells[flagPoint[0]][flagPoint[1]] = this.turn;
			});
		},
		changeTurn() {
			this.turn = 0 - this.turn;
			this.isFinish();
			let stoneColor = this.turn === 1 ? "黒" : "白";
			this.message = stoneColor + "の番です";
		},
		showReverce(y, x) {
			this.showArray = this.reverceFlagArray[y][x];
		},
		AiTurnMove() {
			let AIturn = -1;
			let putStonePosition = null;
			let maxTakeStones = 0;
			for (let i = 0; i <= 5; i++) {
				for (let j = 0; j <= 5; j++) {
					if (
						(this.canTakeStones[i][j] != "s") &
						(maxTakeStones < this.canTakeStones[i][j])
					) {
						maxTakeStones = this.canTakeStones[i][j];
						putStonePosition = [i, j];
					}
				}
			}
			if (maxTakeStones != 0) {
				this.boardCells[putStonePosition[0]][putStonePosition[1]] = AIturn;
				this.reverceStone(putStonePosition[0], putStonePosition[1]);
			} else {
				this.message = "パスしました。";
			}
			this.changeTurn();
			this.updateStoneStates();
		},
		isFinish() {
			let stones = 0;
			for (let i = 0; i <= 5; i++) {
				for (let j = 0; j <= 5; j++) {
					if (this.canTakeStones[i][j] == "s") {
						stones++;
					}
				}
			}
			if (stones == 36) {
				alert("終わり");
			}
		},
	},
	// watch: {
	// 	boardCells: function () {
	// 		this.updateStoneStates();
	// 	},
	// },
	computed: {
		getStonesCount() {
			let black = 0;
			let white = 0;
			let blank = 0;
			for (let i = 0; i <= 5; i++) {
				for (let j = 0; j <= 5; j++) {
					if (this.boardCells[i][j] == 1) {
						black++;
					} else if (this.boardCells[i][j] == -1) {
						white++;
					} else if (this.boardCells[i][j] == 0) {
						blank++;
					}
				}
			}
			return [black, white, blank];
		},
	},
	created() {
		this.updateStoneStates();
	},
};
</script>

<style scoped>
* {
	font-size: 10px;
}
.d-flex {
	display: flex;
}

.d-flex > div {
	margin: 4px;
}
.board {
	border: 1px solid rgb(60, 60, 60);
}

.board td {
	position: relative;
	height: 40px;
	width: 40px;
	background: green;
	border: 1px solid rgb(60, 60, 60);
	text-align: center;
}

.board span.black,
.board span.white {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	-webkit-transform: translate(-50%, -50%);
	-ms-transform: translate(-50%, -50%);
	z-index: 2;
	display: inline-block;
	border-radius: 50%;
	width: 30px;
	height: 30px;
}

.board span.white {
	background: white;
}
.board span.black {
	background: black;
}

.ai-board td {
	height: 20px;
	width: 20px;
	text-align: center;
}

.ai-board {
	border: 1px solid rgb(100, 100, 100);
	margin-bottom: 20px;
}
.ai-board .pointer {
	color: blue;
	border-bottom: blue 1px solid;
	font-weight: bold;
	cursor: pointer;
}
.info {
	min-height: 100px;
}
.msg {
	color: red;
}

.stone > span {
	width: 20px;
	height: 20px;
	border-radius: 50%;
	border: 1px solid black;
}
.stone .white {
	background: white;
}
.stone .black {
	background: black;
}

.blank {
	width: 20px;
	height: 20px;
	border-radius: 0%;
	background: green;
	border: 1px solid rgb(0, 0, 0);
}

.align-center {
	align-items: center;
}
</style>
