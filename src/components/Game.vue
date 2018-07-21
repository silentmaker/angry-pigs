<template>
    <div id="game"></div>
</template>

<script>
import Matter from 'matter-js'
import pigImg from '../assets/pig.png';
import fenceImg from '../assets/fence.png';
import chickenImg from '../assets/chicken.png';

const {
    Engine, Render, Runner, Composites, 
    Events, Constraint, MouseConstraint, 
    Mouse, World, Bodies
} = Matter

export default {
    name: 'game',
    data () {
        return {
            width: 0,
            height: 0,
            itemSize: 0,
            itemScale: 1
        }
    },
    mounted () {
        this.initContainer()
        this.init()

        window.addEventListener("orientationchange", () => history.go(0) )
    },
    methods: {
        initContainer () {
            this.width = window.innerWidth
            this.height = window.innerHeight > window.innerWidth ? window.innerWidth * 0.6 : window.innerHeight
            this.itemSize = Math.max(this.width / 20, 20)
            this.itemScale = this.itemSize / 48
        },
        createPig (x, y) {
            return Bodies.circle(x, y, this.itemSize * 0.5, {
                density: 0.02,
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
            return Bodies.circle(x, y, this.itemSize * 0.5, {
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
                density: 0.01,
                render: {
                    sprite: {
                        texture: chickenImg,
                        xScale: this.itemScale,
                        yScale: this.itemScale
                    }
                }
            })
        },
        createSpace (x, y) {
            return Bodies.circle(x, y, this.itemSize * 0.5, {
                isSensor: true,
                render: { visible: false }
            })
        },
        init () {
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
            const anchor = { x: this.width * 0.2, y: this.height * 0.75 }
            let pig = this.createPig(anchor.x, anchor.y)
            const elastic = Constraint.create({ pointA: anchor, bodyB: pig, stiffness: 0.05 })
            const stack = Composites.stack(this.width * 0.5 , this.height * 0.5, 8, 8, 0, 0, (x, y) => {
                const odds = Math.random()

                if (odds < 0.3) {
                    return this.createChicken(x, y)
                } else if (odds < 0.5) {
                    return this.createFence(x, y)
                } else {
                    return this.createSpace(x, y)
                }
            })

            World.add(world, [ground, stack, pig, elastic])
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
