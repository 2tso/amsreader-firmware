<script>
    import { zeropad, addHours } from './Helpers.js';
    import BarChart from './BarChart.svelte';

    export let json;
    export let sysinfo;

    let config = {};
    let max = 0;
    let min = 0;

    $: {
        let hour = new Date().getUTCHours();
        let i = 0;
        let val = 0;
        let h = 0;
        let d = json["20"] == null ? 2 : 1;
        let yTicks = [];
        let xTicks = [];
        let points = [];
        let cur = new Date();
        addHours(cur, sysinfo.clock_offset);
        for(i = hour; i<24; i++) {
            val = json[zeropad(h++)];
            if(val == null) break;
            xTicks.push({
                label: zeropad(cur.getUTCHours())
            });
            points.push({
                label: val >= 0 ? val.toFixed(d) : '', 
                title: val >= 0 ? val.toFixed(2) + ' ' + json.currency : '',
                value: val >= 0 ? Math.abs(val*100) : 0, 
                label2: val < 0 ? val.toFixed(d) : '', 
                title2: val < 0 ? val.toFixed(2) + ' ' + json.currency : '',
                value2: val < 0 ? Math.abs(val*100) : 0, 
                color: '#7c3aed' 
            });
            min = Math.min(min, val*100);
            max = Math.max(max, val*100);
            addHours(cur, 1);
        };
        for(i = 0; i < 24; i++) {
            val = json[zeropad(h++)];
            if(val == null) break;
            xTicks.push({
                label: zeropad(cur.getUTCHours())
            });
            points.push({
                label: val >= 0 ? val.toFixed(d) : '', 
                value: val >= 0 ? Math.abs(val*100) : 0, 
                label2: val < 0 ? val.toFixed(d) : '', 
                value2: val < 0 ? Math.abs(val*100) : 0, 
                color: '#7c3aed' 
            });
            min = Math.min(min, val*100);
            max = Math.max(max, val*100);
            addHours(cur, 1);
        };

        let range = Math.max(max, Math.abs(min));

        if(min < 0) {
            min = Math.min((range/4)*-1, min);
            let yTicksNum = Math.ceil((Math.abs(min)/range) * 4);
            let yTickDistDown = min/yTicksNum;
            for(i = 1; i < yTicksNum+1; i++) {
                let val = (yTickDistDown*i);
                yTicks.push({
                    value: val,
                    label: (val/100).toFixed(2)
                });
            }
        }

        max = Math.max((range/4), max);
        let xTicksNum = Math.ceil((max/range) * 4);
        let yTickDistUp = max/xTicksNum;
        for(i = 0; i < xTicksNum+1; i++) {
            let val = (yTickDistUp*i);
            yTicks.push({
                value: val,
                label: (val/100).toFixed(2)
            });
        }

        config = {
            title: "Future energy price (" + json.currency + ")",
            padding: { top: 20, right: 15, bottom: 20, left: 35 },
            y: {
                min: min,
                max: max,
                ticks: yTicks
            },
            x: {
                ticks: xTicks
            },
            points: points
        };
    };

</script>

<a href="https://transparency.entsoe.eu/" target="_blank" class="text-xs float-right z-40">Provided by ENTSO-E</a>
<BarChart config={config} />
