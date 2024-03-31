<script lang="ts">
	import kaboom, { type Vec2 } from 'kaboom';

	let score = 0;
	let ended = false;

	const k = kaboom({
		width: 900,
		height: 600
	});

	k.setBackground(k.Color.fromHex('#e2e8ef')); // #e2e8ef

	k.loadSprite('coin', '/assets/coin.png');
	k.loadSprite('princess', '/assets/princess.png');
	k.loadSprite('ramp', '/assets/ramp.png');
	k.loadSprite('tile', '/assets/tile.png');
	k.loadSprite('prince1', '/assets/prince1.png');
	k.loadSprite('prince2', '/assets/prince2.png');
	k.loadSprite('prince3', '/assets/prince3.png');

	k.camPos(new k.Vec2(350, 200));

	const grid = k.add([k.pos(new k.Vec2(0, 0))]);

	const tiles: any = [];

	for (let x = 0; x < 8; x++) {
		for (let y = 0; y < 5; y++) {
			const tile = grid.add([
				'tile',
				k.area(),
				{
					state: 0,
					stateComp: null
				},
				k.sprite('tile'),
				k.scale(0.5),
				k.anchor('center'),
				k.pos(new k.Vec2(x * 100, y * 100))
			]);

			tiles[`${x}_${y}`] = tile;
		}
	}

	k.onMousePress((m) => {
		const allTiles = grid.get('tile');
		for (const tile of allTiles) {
			if (tile.isHovering()) {
				if (m === 'left') {
					tile.state++;
				} else if (m === 'right') {
					tile.state = 0;
				}

				if (tile.state > 2) {
					tile.state = 1;
				}

				if (tile.stateComp) {
					k.destroy(tile.stateComp);
					tile.stateComp = null;
				}

				if (tile.state === 0) {
					return;
				}

				let rotation = 0;

				if (tile.state === 1) {
					rotation = 0;
				} else if (tile.state === 2) {
					rotation = -90;
				}

				tile.stateComp = tile.add([
					'ramp',
					k.area(),
					k.sprite('ramp'),
					k.scale(0.5),
					k.rotate(rotation),
					k.anchor('center'),
					k.pos(0, 0)
				]);
			}
		}
	});

	const princess = tiles['7_2'].add([
		'princess',
		k.area(),
		k.sprite('princess'),
		k.scale(0.6),
		k.anchor('center'),
		k.pos(0, 0)
	]);

	function createPrince() {
		const types = [
			{
				sprite: 'prince1',
				speed: 70
			},
			{
				sprite: 'prince2',
				speed: 90
			},
			{
				sprite: 'prince3',
				speed: 110
			}
		];

		const spots = [
			{
				dir: k.RIGHT,
				poses: [
					new k.Vec2(-50, 0),
					new k.Vec2(-50, 100),
					new k.Vec2(-50, 200),
					new k.Vec2(-50, 300),
					new k.Vec2(-50, 400)
				]
			},
			// {
			// 	dir: k.LEFT,
			// 	poses: [
			// 		new k.Vec2(750, 0),
			// 		new k.Vec2(750, 100),
			// 		// new k.Vec2(750, 200),
			// 		new k.Vec2(750, 300),
			// 		new k.Vec2(750, 400)
			// 	]
			// },
			{
				dir: k.DOWN,
				poses: [
					new k.Vec2(0, -50),
					new k.Vec2(100, -50),
					new k.Vec2(200, -50),
					new k.Vec2(300, -50)
					// new k.Vec2(400, -50)
				]
			},
			{
				dir: k.UP,
				poses: [
					new k.Vec2(0, 450),
					new k.Vec2(100, 450),
					new k.Vec2(200, 450),
					new k.Vec2(300, 450)
					// new k.Vec2(400, 450)
				]
			}
		];

		const spot = spots[Math.floor(Math.random() * spots.length)];
		const pose = spot.poses[Math.floor(Math.random() * spot.poses.length)];
		const type = types[Math.floor(Math.random() * types.length)];

		const prince = grid.add([
			'prince',
			k.sprite(type.sprite),
			k.area(),
			k.scale(0.2),
			k.anchor('center'),
			k.pos(pose.x, pose.y),
			{
				direction: spot.dir,
				speed: type.speed,
				rampState: null as null | number,
				nextPos: null as null | Vec2,
				lastRampId: null as null | string,
				currentRampId: null as null | string
			}
		]);

		prince.onCollide('princess', (princess) => {
			k.destroy(prince);
			score++;
		});

		prince.onCollide('ramp', (ramp) => {
			if (ramp.parent) {
				if (ramp.parent.state === 1) {
					prince.rampState = ramp.parent.state;
				} else if (ramp.parent.state === 2) {
					prince.rampState = ramp.parent.state;
				} else if (ramp.parent.state === 3) {
					prince.rampState = ramp.parent.state;
				} else if (ramp.parent.state === 4) {
					prince.rampState = ramp.parent.state;
				}
			}

			prince.nextPos = ramp.worldPos();
			prince.currentRampId = `${ramp.worldPos().x}_${ramp.worldPos().y}`;
		});

		prince.onUpdate(() => {
			prince.move(prince.direction.scale(prince.speed));

			if (prince.pos.x < -100 || prince.pos.x > 800 || prince.pos.y < -100 || prince.pos.y > 500) {
				alert('Game over. Score: ' + score);
				k.destroy(prince);
				score = 0;

				const allPrincess = grid.get('prince');
				for (const loopPrince of allPrincess) {
					k.destroy(loopPrince);
				}

				const allTiles = grid.get('tile');
				for (const loopTile of allTiles) {
					const allRamps = loopTile.get('ramp');
					for (const loopRamp of allRamps) {
						k.destroy(loopRamp);
					}
				}

				ended = true;
			}

			if (prince.rampState && prince.nextPos) {
				if (prince.lastRampId && prince.lastRampId === prince.currentRampId) {
					return;
				}

				let nextAction = ((_) => false) as (nextPos: Vec2) => boolean;

				if (prince.direction === k.RIGHT) {
					nextAction = (nextPos: Vec2) => {
						return prince.worldPos().x >= nextPos.x;
					};
				} else if (prince.direction === k.UP) {
					nextAction = (nextPos: Vec2) => {
						return prince.worldPos().y <= nextPos.y;
					};
				} else if (prince.direction === k.DOWN) {
					nextAction = (nextPos: Vec2) => {
						return prince.worldPos().y >= nextPos.y;
					};
				} else if (prince.direction === k.LEFT) {
					nextAction = (nextPos: Vec2) => {
						return prince.worldPos().x <= nextPos.x;
					};
				}

				if (nextAction(prince.nextPos)) {
					if (prince.rampState === 0) {
						return;
					}

					prince.lastRampId = prince.currentRampId;

					let nextDirection: Vec2 = k.RIGHT;

					if (prince.direction === k.RIGHT) {
						nextDirection = prince.rampState === 1 ? k.UP : k.DOWN;
					} else if (prince.direction === k.UP) {
						nextDirection = prince.rampState === 1 ? k.RIGHT : k.LEFT;
					} else if (prince.direction === k.LEFT) {
						nextDirection = prince.rampState === 1 ? k.DOWN : k.UP;
					} else if (prince.direction === k.DOWN) {
						nextDirection = prince.rampState === 1 ? k.LEFT : k.RIGHT;
					}

					prince.pos.x = prince.nextPos.x;
					prince.pos.y = prince.nextPos.y;
					prince.direction = nextDirection;
					prince.rampState = null;
					prince.nextPos = null;
					prince.currentRampId = null;
				}
			}
		});
	}

	gameTick();

	function gameTick() {
		createPrince();

		let timeout = 3000 - 100 * score;
		timeout = Math.max(timeout, 2000);

		if (score > 100) {
			timeout = 1500;
		}

		if (score > 250) {
			timeout = 1000;
		}

		setTimeout(() => {
			gameTick();
		}, timeout);
	}

	k.debug.paused = true;
	setTimeout(() => {
	let i = 0;
	while(ended === false && i < 10000) {
		k.debug.stepFrame();
		i++;
		console.log(i);
	}
	}, 100);
</script>

<h1>
	Score <span style="color: rgb(200,200,200)">|</span>
	{score}
</h1>
