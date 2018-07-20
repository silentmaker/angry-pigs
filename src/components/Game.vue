<template>
    <div ref="container" id="game"></div>
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
            height: 0
        }
    },
    mounted () {
        this.width = this.$refs.container.clientWidth
        this.height = this.$refs.container.clientHeight
        this.init()
    },
    methods: {
        createPig (x, y) {
            return Bodies.circle(x, y, 24, {
                density: 0.02,
                render: {
                    sprite: { texture: pigImg }
                }
            })
        },
        createFence (x, y) {
            return Bodies.circle(x, y, 24, {
                isStatic: true,
                render: {
                    sprite: { texture: fenceImg }
                }
            })
        },
        createChicken (x, y) {
            return Bodies.circle(x, y, 24, {
                density: 0.01,
                render: {
                    sprite: { texture: chickenImg }
                }
            })
        },
        createSpace (x, y) {
            return Bodies.circle(x, y, 24, {
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

            // add mouse control
            const mouse = Mouse.create(render.canvas)
            const mouseConstraint = MouseConstraint.create(engine, {
                mouse: mouse,
                constraint: {
                    stiffness: 0.2,
                    render: { visible: false }
                }
            });

            World.add(world, mouseConstraint);
            // keep the mouse in sync with rendering
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
