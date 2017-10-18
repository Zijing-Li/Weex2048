<template>
	<div class="content">
		<div class="header">
			<text>当前朝代：{{dynastyList[currentDynasty].title}}</text>
		</div>
		<div class="game-body" @swipe="swipe">
			<div class="game-row" v-for="(row, rowIndex) in gridList">
				<div class="game-grid"
					 :style="{width:gridSize, height:gridSize}"
					 v-for="(grid, columnIndex) in row">
					<div class="grid-bg" :style="{backgroundColor:dynastyList[grid.dynasty].backgroundColor}"></div>
					<text class="grid-text"
						  v-if="grid.dynasty!=0"
						  :style="{fontSize:dynastyList[grid.dynasty].titleFont}"
					>{{dynastyList[grid.dynasty].title}}</text>
				</div>
			</div>
		</div>
		<div class="footer">
			<text class="reset-button" v-if="gameStatus!=0" @click="resetGame">{{gameStatus==1?'再玩一次':'重来'}}</text>
		</div>
	</div>
</template>

<script>
	const modal = weex.requireModule('modal')
	var timer = null
    export default {
		computed: {
			gridSize: function () {
				return 750.0/this.columnNumber + 'px'
			}
		},
        data: {
		    currentDynasty: 0,
			swiping: false,
			gameStatus: 0, // 0进行中 1赢 2输
			rowNumber: 5,
			columnNumber: 5,
			gridList:[],
			freeGrids:[],
			fontSizeList: [0, 80, 60, 40, 35],
			dynastyList:[
				{
				  	title: '',
					backgroundColor: '#CCBFB2'
				},
			    {
			        title: '夏',
					backgroundColor: '#EDE2DB'
				},
				{
					title: '商',
					backgroundColor: '#EADDC6'
				},
				{
					title: '周',
					backgroundColor: '#FAEBD7'
				},
				{
					title: '秦',
					backgroundColor: '#F5F5DC'
				},
				{
					title: '汉',
					backgroundColor: '#FFE4C4'
				},
				{
					title: '三国',
					titleColor: 'white',
					backgroundColor: '#DEB887'
				},
				{
					title: '晋',
					backgroundColor: '#D2691E'
				},
				{
					title: '南北朝',
					backgroundColor: '#FF7F50'
				},
				{
					title: '隋',
					backgroundColor: '#B8860B'
				},
				{
					title: '唐',
					backgroundColor: '#BDB76B'
				},
				{
					title: '五代十国',
					backgroundColor: '#FF8C00'
				},
				{
					title: '宋',
					backgroundColor: '#E9967A'
				},
				{
					title: '元',
					backgroundColor: '#B22222'
				},
				{
					title: '明',
					backgroundColor: '#FF69B4'
				},
				{
					title: '清',
					backgroundColor: '#CD5C5C'
				},
				{
					title: '民国',
					backgroundColor: '#FF4500'
				},
				{
					title: '大天朝',
					backgroundColor: '#FF0000'
				}
			]
		},
        methods: {
			failed: function () {
			    this.gameStatus = 2
				toast('你输了！', 3)
			},
			success: function () {
				this.gameStatus = 1
				toast('你赢了！', 3)
			},
		    titleSize: function (title) {
				if (title.length >= this.fontSizeList.length) {
				    return this.fontSizeList[this.fontSizeList.length-1]
				}
				return this.fontSizeList[title.length]
			},
            swipe: function (ret) {
		        if (this.swiping || this.gameStatus != 0) {
		            return
				}
				clearInterval(timer)
				timer = null
                switch (ret.direction) {
					case 'up':
					    if (!this.canSwipeColumn(true)) {
					        return
						}
						this.swiping = true
					    this.swipeColumn(true)
					    break
					case 'left':
						if (!this.canSwipeRow(true)) {
						    return
						}
						this.swiping = true
						this.swipeRow(true)
					    break
					case 'down':
					    if (!this.canSwipeColumn(false)) {
					        return
						}
						this.swiping = true
					    this.swipeColumn(false)
					    break
					case 'right':
					    if (!this.canSwipeRow(false)) {
					        return
						}
						this.swiping = true
					    this.swipeRow(false)
					    break
					default:
					    break
				}
			},
			swipeRow: function (isLeft) {
				for (var rowIndex in this.gridList) {
				    var row = this.gridList[rowIndex]
					var swipeGrid = null
				    for (var i=isLeft?0:row.length-1;;) {
				        var grid = row[i]
						if (swipeGrid!=null) {
						    if (swipeGrid.dynasty==0 && grid.dynasty!=0) {
								swipeGrid.dynasty=grid.dynasty
								grid.dynasty=0
								this.updateGrid(grid)
								this.updateGrid(swipeGrid)
							} else if (swipeGrid.dynasty!=0 && grid.dynasty!=0 && swipeGrid.dynasty==grid.dynasty) {
								swipeGrid.dynasty++
								if (this.currentDynasty < swipeGrid.dynasty) {
								    this.currentDynasty = swipeGrid.dynasty
								}
								grid.dynasty=0
								this.updateGrid(grid)
								this.updateGrid(swipeGrid)
							}
						}
						swipeGrid = grid
						if (isLeft) {
							i++
							if (i>=row.length) {
								break
							}
						} else {
							i--
							if (i<0) {
								break
							}
						}
					}
				}
				if (this.canSwipeRow(isLeft)) {
				    if (timer == null) {
						var self = this
						timer = setInterval(function () {
							self.swipeRow(isLeft)
						}, 50)
					}
				} else {
				    clearInterval(timer)
					timer = null
					// 随机添加一个朝代
					this.randomDynasty(1, 1, 1)
					this.swiping = false
				}
			},
			canSwipeRow: function (isLeft) {
				for (var rowIndex in this.gridList) {
					var row = this.gridList[rowIndex]
					var swipeGrid = null
					for (var i=isLeft?0:row.length-1;;) {
						var grid = row[i]
						if (swipeGrid!=null) {
							if (swipeGrid.dynasty==0 && grid.dynasty!=0) {
								return true
							} else if (swipeGrid.dynasty!=0 && grid.dynasty!=0 && swipeGrid.dynasty==grid.dynasty) {
								return true
							}
						}
						swipeGrid = grid
						if (isLeft) {
							i++
						    if (i>=row.length) {
						        break
							}
						} else {
						    i--
							if (i<0) {
						        break
							}
						}
					}
				}
				return false
			},
			swipeColumn: function (isUp) {
				for (var columnIndex=0; columnIndex<this.columnNumber; columnIndex++) {
					var swipeGrid = null
					for (var rowIndex=isUp?0:this.rowNumber-1;;) {
						var grid = this.gridList[rowIndex][columnIndex]
						if (swipeGrid!=null) {
							if (swipeGrid.dynasty==0 && grid.dynasty!=0) {
								swipeGrid.dynasty=grid.dynasty
								grid.dynasty=0
								this.updateGrid(grid)
								this.updateGrid(swipeGrid)
							} else if (swipeGrid.dynasty!=0 && grid.dynasty!=0 && swipeGrid.dynasty==grid.dynasty) {
								swipeGrid.dynasty++
								if (this.currentDynasty < swipeGrid.dynasty) {
									this.currentDynasty = swipeGrid.dynasty
								}
								grid.dynasty=0
								this.updateGrid(grid)
								this.updateGrid(swipeGrid)
							}
						}
						swipeGrid = grid
						if (isUp) {
							rowIndex++
							if (rowIndex>=this.rowNumber) {
								break
							}
						} else {
							rowIndex--
							if (rowIndex<0) {
								break
							}
						}
					}
				}
				if (this.canSwipeColumn(isUp)) {
					if (timer == null) {
						var self = this
						timer = setInterval(function () {
							self.swipeColumn(isUp)
						}, 50)
					}
				} else {
					clearInterval(timer)
					timer = null
					// 随机添加一个朝代
					this.randomDynasty(1, 1, 1)
					this.swiping = false
				}
			},
			canSwipeColumn: function (isUp) {
		        for (var columnIndex=0; columnIndex<this.columnNumber; columnIndex++) {
					var swipeGrid = null
		            for (var rowIndex=isUp?0:this.rowNumber-1;;) {
						var grid = this.gridList[rowIndex][columnIndex]
						if (swipeGrid!=null) {
							if (swipeGrid.dynasty==0 && grid.dynasty!=0) {
								return true
							} else if (swipeGrid.dynasty!=0 && grid.dynasty!=0 && swipeGrid.dynasty==grid.dynasty) {
								return true
							}
						}
						swipeGrid = grid
						if (isUp) {
							rowIndex++
							if (rowIndex>=this.rowNumber) {
								break
							}
						} else {
							rowIndex--
							if (rowIndex<0) {
								break
							}
						}
					}
				}
				return false
			},
			updateGrid: function (grid) {
				this.gridList[grid.row].splice(grid.column, 1, grid)
				if (grid.dynasty==0) {
				    if (!contains(this.freeGrids, grid)) {
				        this.freeGrids.push(grid)
					}
				} else {
				    var index = objIndex(this.freeGrids, grid)
					if (index != -1) {
						this.freeGrids.splice(index, 1)
					}
				}
			},
			// 随机添加朝代，count 数量， minDynasty 最小朝代， maxDynasty 最大朝代
			randomDynasty: function (count, minDynasty, maxDynasty) {
			    if (this.currentDynasty == this.dynastyList.length-1) {
			        this.success()
					return
				}
				for (var i=0; i<count; i++) {
					var grid = this.freeGrids[Math.floor(Math.random()*this.freeGrids.length)]
					var dynasty = minDynasty + Math.floor(Math.random()*(maxDynasty - minDynasty + 1))
					grid.dynasty = dynasty
					this.freeGrids.splice(objIndex(this.freeGrids, grid), 1)
				}
				if (this.freeGrids.length==0
					&& this.canSwipeRow(true)==false
					&& this.canSwipeRow(false)==false
					&& this.canSwipeColumn(true)==false
					&& this.canSwipeColumn(false)==false) {
				    this.failed()
				}
			},
			resetGame: function () {
				this.gameStatus = 0
				this.freeGrids = []
				this.currentDynasty = 0
				for (var i=0; i<this.rowNumber; i++) {
					var row
					if (i<this.gridList.length) {
						row = this.gridList[i]
						this.gridList.splice(i, 1, row)
					} else  {
						row = []
						this.gridList.push(row)
					}
					for (var j=0; j<this.columnNumber; j++) {
						var grid
						if (j<row.length) {
							grid = row[j]
						} else {
							grid = {}
							row.push(grid)
						}
						grid.row = i
						grid.column = j
						grid.dynasty = 0
						this.freeGrids.push(grid)
					}
				}
				this.randomDynasty(2, 1, 1)
			}
		},
        created: function () {
			for (var i in this.dynastyList) {
			    var dynasty = this.dynastyList[i]
				dynasty.titleFont = this.titleSize(dynasty.title)
			}

			this.resetGame()
        }
    }
	function contains(arr, obj) {
		for (var i in arr) {
			if (arr[i] === obj) {
				return true
			}
		}
		return false
	}
	function objIndex(arr, obj) {
		for (var i in arr) {
			if (arr[i] === obj) {
				return i
			}
		}
		return -1
	}
	function toast(text, duration) {
		modal.toast({
			message: text,
			duration: duration
		})
	}
</script>

<style scoped>
	.content {
		background-color: white;
		flex-direction: column;
	}
	.header {
		flex: 1;
	}
	.footer {
		height: 60wx;
		flex-direction: row;
		justify-content: center;
		align-items: center;
	}
	.game-body {
		width: 750px;
		height: 750px;
		flex-direction: column;
	}
	.game-row {
		flex-direction: row;
	}
	.game-grid {
		flex-direction: row;
		justify-content: center;
		align-items: center;
	}
	.grid-bg {
		position: absolute;
		left: 2wx;
		top: 2wx;
		right: 2wx;
		bottom: 2wx;
	}
	.grid-text {
		color: white;
	}
	.reset-button {
		font-size: 30wx;
		color: orange;
	}
</style>
