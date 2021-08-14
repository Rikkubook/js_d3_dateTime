<template>
  <div id="app"></div>
</template>

<script>
import * as d3 from "d3";

export default {
  name: "App",
  mounted() {
    // 原始資料，以小時為單位
    let row_data = [
      {
        time: 0,
        thing: "工作",
      },
      {
        time: 1,
        thing: "工作",
      },
      {
        time: 2,
        thing: "工作",
      },
      {
        time: 3,
        thing: "工作",
      },
      {
        time: 4,
        thing: "工作",
      },
      {
        time: 5,
        thing: "工作",
      },
      {
        time: 6,
        thing: "工作",
      },
      {
        time: 7,
        thing: "睡覺",
      },
      {
        time: 8,
        thing: "睡覺",
      },
      {
        time: 9,
        thing: "睡覺",
      },
      {
        time: 10,
        thing: "睡覺",
      },
      {
        time: 11,
        thing: "睡覺",
      },
      {
        time: 12,
        thing: "工作",
      },
      {
        time: 13,
        thing: "工作",
      },
      {
        time: 14,
        thing: "睡覺",
      },
      {
        time: 15,
        thing: "睡覺",
      },
      {
        time: 16,
        thing: "做專案",
      },
      {
        time: 17,
        thing: "做專案",
      },
      {
        time: 18,
        thing: "做專案",
      },
      {
        time: 19,
        thing: "做專案",
      },
      {
        time: 20,
        thing: "吃飯",
      },
      {
        time: 21,
        thing: "吃飯",
      },
      {
        time: 22,
        thing: "休息",
      },
      {
        time: 23,
        thing: "休息",
      },
    ];

    // 動態產生用的類別
    let catas = ["工作", "睡覺", "專案", "吃東西", "散步"];

    // 動態產生
    let last = null;

    function generate_day_date() {
      return d3.range(0, 24).map(
        // o 0~23
        (o) => {
          if (!last || Math.random() > 0.5) {
            // 美產生一個有0.5機率變換
            last = catas[parseInt(Math.random() * catas.length)];
          }
          return {
            time: o,
            thing: last,
          };
        }
      );
    }

    row_data = generate_day_date();

    let newData = row_data
      .map((o) => o.thing) // 抽出需要的
      .filter((d, i, arr) => arr.indexOf(d) === i); // 篩選單一
    // console.log(row_data);

    // 將縮減包成函式
    function reduce_time(arr) {
      let joined_data = arr.reduce((all, el) => {
        var last_el = all.slice(-1)[0]; // 最後一個時段 物件
        var last_thing = last_el ? last_el.thing : null;
        if (last_thing == el.thing) {
          last_el.end_hour = el.time;
          return all;
        } else {
          return all.concat({
            thing: el.thing,
            start_hour: el.time,
            end_hour: el.time,
          });
        }
      }, []); // 最後希望 [{thing: 工作, start_hour: 9, end_hour: 11},...]
      return joined_data;
    }

    let data = d3.range(1, 8); // 有7天
    // d3內建的函式
    // console.log(data);
    let svg = d3
      .select("body")
      .append("svg") // 每天寬高
      .attr("width", 700)
      .attr("height", 700);

    let data_week = d3.range(0, 7).map(() => reduce_time(generate_day_date()));
    console.log(data_week);
    let hour_width = 50; // 每小時寬度
    let hour_height = 30; // 每小時高度
    // let colorize = d3.scaleLinear().domain([0, 24]).range(["#fff", "#000"]);
    // 定義一個色票檔 會分24格並相對應漸層
    let colorize = d3.scaleOrdinal().range(d3.schemePastel1);
    // 對應數組上不同色

    let day_group = svg
      .selectAll("g")
      .data(data_week)
      .enter()
      .append("g")
      .attr("transform", (d, i) => `translate(${i * 100})`);

    // 添加底色
    let hour_rect = day_group
      .selectAll("rect.hour")
      .data((d) => d)
      .enter()
      .append("rect")
      .attr("class", "hour")
      //初始位置
      .attr("y", (d, i) => d.start_hour * hour_height)
      .attr("width", hour_width)
      .attr("height", (d, i) => (d.end_hour - d.start_hour + 1) * hour_height)
      .attr("stroke", "black")
      .attr("fill", (d, i) => colorize(newData.indexOf(d.thing)));

    // 添加文字
    let hour_text = day_group
      .selectAll("text.hour")
      .data((d) => d)
      .enter()
      .append("text")
      .attr("class", "text")
      .attr("y", (d, i) => d.start_hour * hour_height + 20)
      .attr("x", (d, i) => 7)
      .attr("fill", "#222")
      .text((d, i) => d.thing);
  },
};
</script>

<style>
.text {
  /* svg align 起始點 */
  alignment-baseline: hanging;
  font-size: 12px;
}
</style>
