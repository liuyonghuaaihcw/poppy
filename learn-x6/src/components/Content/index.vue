<template>
    <div id="x6_container" class="home"></div>
</template>

<script>
import { Graph, Shape } from '@antv/x6';
import { mapMutations } from "vuex";
import { Mouse } from "@/events/mouse";
import { Keyboard } from "@/events/keyboard";

export default {
    name: "index",
    data() {
        return {
            graph: null,
            graphOption: {
                grid: {
                    size: 1, // 网格大小10px
                    visible: false,
                    type: 'doubleMesh',
                    args: [
                        {
                            color: '#E7E8EA', // 主网格线颜色
                            thickness: 1, // 主网格线宽度
                        },
                        {
                            color: '#CBCED3', // 次网格线颜色
                            thickness: 1, // 次网格线宽度
                            factor: 5, // 主次网格线间隔
                        },
                    ],
                },
                // 画布平移
                panning: {
                    enabled: true,
                    eventTypes: [ 'leftMouseDown', 'rightMouseDown', 'mouseWheel' ],
                    modifiers: 'ctrl',
                },
                mousewheel: {
                    enabled: true,
                    zoomAtMousePosition: true,
                    modifiers: 'ctrl',
                    minScale: 0.5,
                    maxScale: 3,
                },
                // 默认的连线规则
                connecting: {
                    anchor: 'center',
                    connectionPoint: 'anchor',
                    allowBlank: false,
                    allowLoop: false,
                    allowNode: false,
                    highlight: true,
                    snap: {
                        radius: 20,
                    },
                    // 连接的过程中创建新的边
                    createEdge() {
                        return new Shape.DoubleEdge({
                            attrs: {
                                line: {
                                    stroke: '#000',
                                    strokeWidth: 1,
                                    targetMarker: {
                                        name: 'classic',
                                        args: {
                                            width: 12,
                                            height: 8,
                                            stroke: "#000000",
                                            fill: "#000000",
                                            strokeWidth: 1
                                        }
                                    },
                                },
                                outline: {
                                    stroke: 'transparent',
                                    strokeWidth: 3,
                                },
                                text: {
                                    fontSize: 14,
                                    fontFamily: '微软雅黑',
                                    fontColor: '#000000'
                                }
                            },
                            router: {
                                name: 'manhattan',
                            },
                            connector: {
                                name: 'rounded'
                            }
                        });
                    }
                },
                highlighting: {
                    magnetAdsorbed: {
                        name: 'stroke',
                        args: {
                            attrs: {
                                fill: '#D06269',
                                stroke: '#D06269',
                            },
                        },
                    },
                },
                resizing: {
                    enabled: true,
                    preserveAspectRatio: node => node.prop('preserveAspectRatio') // 节点设置了锁定宽高比时才进行等比例缩放
                },
                rotating: true,
                selecting: {
                    enabled: true,
                    rubberband: true,
                    showNodeSelectionBox: true,
                    rubberEdge: true
                },
                snapline: true,
                keyboard: true,
                history: true,
                clipboard: true,
            },
            size: {
                width: 0,
                height: 0
            }
        };
    },
    mounted() {
        this.getContainerSize();
        // this.initGraph().fromJSON(data);
        this.initGraph();
        this.resize();
        this.initEvent();
        window.graph = this.graph
    },
    methods: {
        ...mapMutations('app', [
            'setIsInit',
            'setGraph',
            'setConfigType',
            'setCellId',
            'setSelectedCell'
        ]),
        // init
        initGraph() {
            const graph = new Graph({
                container: document.getElementById('x6_container'),
                ...this.size,
                ...this.graphOption,
                minimap: {
                    enabled: true,
                    container: document.getElementById('minimap'),
                    width: 198,
                    height: 198,
                    padding: 10,
                },
            });
            this.graph = graph;
            this.setIsInit(true);
            this.setGraph(graph);
            console.log(graph);
            return graph;
        },
        // 获取画布区域的宽高
        getContainerSize() {
            const { width, height } = document.querySelector('#content').getBoundingClientRect();
            this.size.width = width;
            this.size.height = height;
        },
        // 监听窗口变化，画布自适应
        resize() {
            window.addEventListener('resize', () => {
                this.getContainerSize();
                const { width, height } = this.size;
                this.graph.resize(width, height);
            });
        },
        // 初始化事件
        initEvent: function () {
            const mouse = new Mouse(this.graph);
            const keyboard = new Keyboard(this.graph);
            mouse.bind();
            keyboard.bind();
            mouse._getClickType((cellId, selectedCell, configType) => {
                this.setConfigType(configType);
                this.setSelectedCell(selectedCell);
                this.setCellId(cellId);
            });
        },

    }
};
</script>

<style scoped>
#x6_container {
    width: 100%;
    height: 100%;
}
</style>
