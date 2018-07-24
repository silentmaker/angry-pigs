<template>
    <div id="game"></div>
</template>

<script>
import Matter from 'matter-js'
import pigImg from '../assets/pig.png';
import fenceImg from '../assets/fence.png';
import chickenImg from '../assets/chicken.png';

const {
    Engine, Render, Runner, Composites, Composite, Common, 
    Events, Constraint, MouseConstraint, Mouse, World, Bodies
} = Matter

export default {
    name: 'game',
    data () {
        return {
            width: 0,
            height: 0,
            itemSize: 0,
            itemScale: 1,
            columns: 12,
            isNormal: true
        }
    },
    mounted () {
        this.initContainer()
        this.initGame()

        window.addEventListener("orientationchange", () => {
            this.isNormal = window.orientation === 0
        })
    },
    methods: {
        initContainer () {
            this.width = window.innerWidth
            this.height = window.innerHeight
            this.itemSize = this.width / this.columns
            this.itemScale = this.itemSize / 48
            this.isNormal = window.orientation === 0
        },
        createPig (x, y) {
            return Bodies.circle(x, y, this.itemSize * 0.5, {
                density: 0.05,
                render: {
                    sprite: {
                        texture: pigImg,
                        xScale: this.itemScale,
                        yScale: this.itemScale
                    }
                }
            })
        },
        createFence (x, y) {
            return Bodies.rectangle(x, y, this.itemSize, this.itemSize, {
                isStatic: true,
                render: {
                    sprite: {
                        texture: fenceImg,
                        xScale: this.itemScale,
                        yScale: this.itemScale
                    }
                }
            })
        },
        createChicken (x, y) {
            return Bodies.circle(x, y, this.itemSize * 0.5, {
                density: 0.02,
                render: {
                    sprite: {
                        texture: chickenImg,
                        xScale: this.itemScale,
                        yScale: this.itemScale
                    }
                }
            })
        },
        initGame () {
            const engine = Engine.create()
            const { world } = engine
            const runner = Runner.create()
            const render = Render.create({
                element: document.getElementById('game'),
                engine: engine,
                options: {
                    width: this.width,
                    height: this.height,
                    background: '#465A63',
                    wireframes: false
                }
            })

            Render.run(render)
            Runner.run(runner, engine)

            const ground = Bodies.rectangle(this.width / 2, this.height, this.width, 1, { isStatic: true })
            const leftWall = Bodies.rectangle(0, this.height / 2, 1, this.height, { isStatic: true })
            const rightWall = Bodies.rectangle(this.width, this.height / 2, 1, this.height, { isStatic: true })
            const anchor = { x: this.width * 0.5, y: this.height * 0.8 }
            let pig = this.createPig(anchor.x, anchor.y)
            const elastic = Constraint.create({ pointA: anchor, bodyB: pig, stiffness: 0.1 })
            const targets = []
            for (let i = 1; i < this.columns; i += 2) {
                const comp = Composite.create()
                const random = Math.random() * 10 + 0.5
                Composite.add(comp, this.createChicken(i * this.itemSize, random * this.itemSize))
                Composite.add(comp, this.createFence(i * this.itemSize, (random + 1) * this.itemSize))
                targets.push(comp)
            }
            World.add(world, [ground, leftWall, rightWall, pig, elastic].concat(targets))

            Events.on(engine, 'afterUpdate', () => {
                if (mouseConstraint.mouse.button === -1 && (pig.position.x > anchor.x + 10 || pig.position.y < anchor.y - 10)) {
                    pig = this.createPig(anchor.x, anchor.y)
                    elastic.bodyB = pig
                    World.add(world, pig)
                }
            })

            const mouse = Mouse.create(render.canvas)
            const mouseConstraint = MouseConstraint.create(engine, {
                mouse: mouse,
                constraint: {
                    stiffness: 0.2,
                    render: { visible: false }
                }
            });

            World.add(world, mouseConstraint);
            render.mouse = mouse;
            Render.lookAt(render, {
                min: { x: 0, y: 0 },
                max: { x: this.width, y: this.height }
            });
        }
    }
}
</script>

<style>
#game {
    position: absolute;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
}
</style>
