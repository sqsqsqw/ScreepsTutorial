# Screeps中文笔记

## 中文文档

[中文文档](https://screeps-cn.github.io)，由国内玩家自发维护，包含 api 文档。已完全汉化。

在玩Screeps之前，尽量看一遍官方文档。然后通过官方的[新手教程](https://screeps.com/a/#!/sim/tutorial)。如果全部通过，说明你很适合这个游戏，那么就开始搭建环境吧。

## 新手教程

### 第一关：游戏UI和基本脚本

- [find函数](https://screeps-cn.github.io/api/#Room.find)
- [moveTo函数](https://screeps-cn.github.io/api/#Creep.moveTo)
- [harvest函数](https://screeps-cn.github.io/api/#Creep.harvest) 
- [spawnCreep函数](https://screeps-cn.github.io/api/#StructureSpawn.spawnCreep)

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
//call module role.harvester
var roleHarvester = require('role.harvester');

module.exports.loop = function () {

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        roleHarvester.run(creep);
    }
}
```
```javascript
//new module named role.harvester
var roleHarvester = {
    /** @param {Creep} creep **/
    run: function(creep) {
	    if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }
        else {
            if(creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                creep.moveTo(Game.spawns['Spawn1']);
            }
        }
	}
};
module.exports = roleHarvester;
```

### 第er关：升级控制器


```javascript
// 创建一个署名为 Upgrader 的 creep，并与 Harvester 进行区分
Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Upgrader1' );
```

但是这样创建只是在creep的名称上进行区分，需要对creep进行角色区分，则需要使用接下来的语句

```javascript
Game.creeps['Harvester1'].memory.role = 'harvester';
Game.creeps['Upgrader1'].memory.role = 'upgrader';
```

```javascript
//new module named role.upgrader
var roleUpgrader = {

    /** @param {Creep} creep **/
    run: function(creep) {
	    if(creep.store[RESOURCE_ENERGY] == 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }
        else {
            if(creep.upgradeController(creep.room.controller) == ERR_NOT_IN_RANGE) {
                creep.moveTo(creep.room.controller);
            }
        }
	}
};

module.exports = roleUpgrader;
```
```javascript
var roleHarvester = require('role.harvester');
var roleUpgrader = require('role.upgrader');

module.exports.loop = function () {

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'upgrader') {
            roleUpgrader.run(creep);
        }
    }
}
```

### 第three关：构建建筑物

```javascript

Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Builder1',{ memory: { role: 'builder' } } );
```

```javascript

var roleBuilder = {
//new module named role.builder
    /** @param {Creep} creep **/
    run: function(creep) {

	    if(creep.memory.building && creep.store[RESOURCE_ENERGY] == 0) {
            creep.memory.building = false;
            creep.say('🔄 harvest');
	    }
	    if(!creep.memory.building && creep.store.getFreeCapacity() == 0) {
	        creep.memory.building = true;
	        creep.say('🚧 build');
	    }

	    if(creep.memory.building) {
	        var targets = creep.room.find(FIND_CONSTRUCTION_SITES);
            if(targets.length) {
                if(creep.build(targets[0]) == ERR_NOT_IN_RANGE) {
                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
                }
            }
	    }
	    else {
	        var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
            }
	    }
	}
};

module.exports = roleBuilder;
```

```javascript
var roleHarvester = require('role.harvester');
var roleBuilder = require('role.builder');

module.exports.loop = function () {

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'builder') {
            roleBuilder.run(creep);
        }
    }
}
```

## 使用vscode

