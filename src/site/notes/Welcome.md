---
{"dg-publish":true,"dg-home":"true","permalink":"/welcome/","tags":["gardenEntry"],"dgPassFrontmatter":true,"noteIcon":"","created":"2024-12-24T13:56:28.000+08:00","updated":"2025-04-09T14:38:30.576+08:00"}
---



``

``` dataviewjs

const calendarData = {
    colors: {
        green: ["#d6f5d6", "#a8e6a3", "#72d872", "#44c144", "#238823"]
    },
    
    entries: []
};

// 统计每个日期写了多少篇笔记
let dateMap = new Map();

for (let page of dv.pages().where(p => p.file && p.file.cday)) {
    const date = page.file.cday.toFormat("yyyy-MM-dd");
    dateMap.set(date, (dateMap.get(date) || 0) + 1);
}

// 生成 entries（无内容提示，无hover）
calendarData.entries = Array.from(dateMap.entries()).map(([date, count]) => ({
    date,
    intensity: count,
    color: "green"
}));

renderHeatmapCalendar(this.container, calendarData);



```
