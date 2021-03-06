<template>
    <div>
        <geometry-element
                selectable
                :movable="editMode"
                :resizable="editMode"
                connectable
                :deletable=editMode
                :id.sync="value.elementView.id"
                :x.sync="value.elementView.x"
                :y.sync="value.elementView.y"
                :width.sync="value.elementView.width"
                :height.sync="value.elementView.height"
                :angle.sync="value.elementView.angle"
                v-on:selectShape="selectedActivity"
                v-on:deSelectShape="deSelectedActivity"
                v-on:dblclick="showProperty"
                v-on:rotateShape="onRotateShape"
                v-on:labelChanged="onLabelChanged"
                v-on:addedToGroup="onAddedToGroup"
                v-on:removeShape="onRemoveShape(value)"
                :label.sync="name"
                :_style="{
                'label-angle':value.elementView.angle,
                'font-weight': 'bold','font-size': '16'
                }"
                v-on:contextmenu.prevent.stop="handleClick($event)"
        >

            <!--v-on:dblclick="$refs['dialog'].open()"-->
            <geometry-rect
                    :_style="{
                        'fill-r': 1,
                        'fill-cx': .1,
                        'fill-cy': .1,
                        'stroke-width': 1.4,
                        'stroke': '#F1A746',
                        fill: '#F1A746',
                        'fill-opacity': 1,
                        r: '1'
                    }"
            ></geometry-rect>

            <sub-controller
                    :image="'subprocess.png'"
                    @click.prevent.stop="handleClick($event)"
            ></sub-controller>

            <sub-elements>
                <!--title-->
                <text-element
                        :sub-width="'100%'"
                        :sub-height="25"
                        :sub-top="0"
                        :sub-left="0"
                        :text="'DestinationRule'">
                </text-element>
            </sub-elements>
        </geometry-element>

        <property-panel
                v-if="openPanel"
                v-model="value"
                :img="imgSrc">
        </property-panel>

        <vue-context-menu
            :elementId="value.elementView.id"
            :options="menuList"
            :ref="'vueSimpleContextMenu'"
            @option-clicked="optionClicked">
        </vue-context-menu>
    </div>
</template>

<script>
    import Element from '../Kube-Element'
    import PropertyPanel from './DestinationRulePropertyPanel'

    export default {
        mixins: [Element],
        name: 'destinationRule',
        components: {
            "property-panel": PropertyPanel
        },
        props: {},
        computed: {
            imgSrc() {
                return `${ window.location.protocol + "//" + window.location.host}/static/image/symbol/kubernetes/istio/istio.svg`
            },
            defaultStyle() {
                return {}
            },
            className() {
                return 'DestinationRule'
            },
            createNew(elementId, x, y, width, height) {
                return {
                    _type: this.className(),
                    name: '',
                    namespace: '',
                    namespaceId: '',
                    elementView: {
                        '_type': this.className(),
                        'id': elementId,
                        'x': x,
                        'y': y,
                        'width': width,
                        'height': height,
                        'style': JSON.stringify({}),
                        'angle': 0,
                    },
                    object: {
                        "apiVersion": "networking.istio.io/v1alpha3",
                        "kind": "DestinationRule",
                        "metadata": {
                            "name": ""
                        },
                        "spec": {
                            "host": "",
                            "subsets": [
                                {
                                    "name": "version1",
                                    "labels": {
                                        "version": "v1"
                                    }
                                }
                            ],
                            "trafficPolicy": {
                                "connectionPool": {
                                    "http": {
                                        "http1MaxPendingRequests": 1,
                                        "maxRequestsPerConnection": 1
                                    },
                                    "tcp": {
                                        "maxConnections": 1
                                    }
                                },
                                "outlierDetection": {
                                    "baseEjectionTime": "30s",
                                    "consecutiveErrors": 5,
                                    "interval": "10s",
                                    "maxEjectionPercent": 10
                                }
                            }
                        }
                    },
                    connectableType: [ "Service" ],
                    outboundService: "",
                }
            },
            name() {
                try {
                    return this.value.object.metadata.name;
                } catch (e) {
                    return "Untitled";
                }
            },
            namespace: {
                get: function () {
                    return this.value.object.metadata.namespace
                },
                set: function (newVal) {
                    this.value.object.metadata.namespace = newVal
                }
            },
            outboundServiceName() {
                try{
                    return this.value.outboundService.object.metadata.name
                } catch(e) {
                    return ""
                }
            }
        },
        data: function () {
            return {};
        },
        created: function () {
        },
        mounted: function () {
            var me = this;

            this.$EventBus.$on(`${me.value.elementView.id}`, function (obj) {

                if(obj.action=="addRelation" && obj.element && obj.element.targetElement && obj.element.targetElement._type == "Service") {
                    me.value.outboundService = obj.element.targetElement;
                }
                if(obj.action=="deleteRelation" && obj.element && obj.element.targetElement && obj.element.targetElement._type == "Service") {
                    me.value.outboundService = null;
                }

                if(obj.action=="addRelation" && obj.element && obj.element.sourceElement && obj.element.sourceElement._type == "VirtualService") {
                    me.value.routeType = 'weight';
                    me.value.weight = 100;
                }
                if(obj.action=="deleteRelation" && obj.element && obj.element.sourceElement && obj.element.sourceElement._type == "VirtualService") {
                    delete me.value.routeType;
                    delete me.value.weight;
                }

            })

        },
        watch: {
            name(appName) {
                this.value.name = appName
            },
            outboundServiceName(val) {
                var me = this;
                me.value.object.spec.subsets[0].labels.app = "";
                me.value.object.spec.host = me.value.outboundService.object.metadata.name;
                me.value.object.spec.subsets[0].labels.app = me.value.outboundService.object.spec.selector.app;
            }
        },
        methods: {
        },
    }
</script>

<style>

</style>