# Screeps中文笔记

## 中文文档

[中文文档](https://screeps-cn.github.io)，由国内玩家自发维护，包含 api 文档。已完全汉化。

在玩Screeps之前，尽量看一遍官方文档。然后通过官方的[新手教程](https://screeps.com/a/#!/sim/tutorial)。如果全部通过，说明你很适合这个游戏，那么就开始搭建环境吧。

## 新手教程

### 第一关：游戏UI和基本脚本

```javascript
// spawnCreep 函数用于生成Creep，并规定对应的属性。
Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Harvester1' );
```

```javascript
/*
    loop 用于在每个tick时间段结束的时刻运行对应的函数
    find(type, [opts]) 函数返回一个房间内指定类型的单位的数组
    FIND_SOURCES 是一个常量，用于表示Source单位的类型
    moveTo(x, y, [opts])
          (target, [opts]) 在同一房间内找到目标的最佳路径，然后移动到该空间。如果目标位于另一个房间中，则相应的出口将用作目标。
    harvest(target) 从矿物和沉积物中获取能源。目标必须在Screep附近的正方形上。
*/
module.exports.loop = function () {
    var creep = Game.creeps['Harvester1'];
    var sources = creep.room.find(FIND_SOURCES);
    if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
        creep.moveTo(sources[0]);
    }
}
```

- [find函数](https://docs.screeps.com/api/#Room.find)
- [moveTo函数](https://docs.screeps.com/api/#Creep.moveTo)
- [harvest函数](https://docs.screeps.com/api/#Creep.harvest)

```javascript
module.exports.loop = function () {
    var creep = Game.creeps['Harvester1'];

    if(creep.store.getFreeCapacity() > 0) {
        var sources = creep.room.find(FIND_SOURCES);
        if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
            creep.moveTo(sources[0]);
        }
    }
    else {
        if( creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE ) {
            creep.moveTo(Game.spawns['Spawn1']);
        }
    }
}
```

## 使用vscode

