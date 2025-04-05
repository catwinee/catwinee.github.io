---
title: "记录Wandb首次PR：Wandb Sweeper支持参数追加与覆盖"
published: 2023-11-15
tags: [Hydra, Wandb, 机器学习工作流, 配置管理]
category: 技术实践
draft: false
---

# 参数动态管理新方案

最近在开源社区贡献的[#1856 PR](https://github.com/wandb/wandb/pull/1856)实现了Hydra配置与Wandb Sweeper的深度集成，通过`args_append_hydra`和`args_override_hydra`两个新指令，为机器学习实验管理带来更灵活的配置控制能力。

## 思路
提交这一PR的灵感来自于上个学期的科研训练。当时使用`Hydra-Lightning`模板配合`wandb`进行训练。
然而在使用`sweep`功能自动化调参时，发现`wandb`并不原生支持给Hydra(OmegaConf)进行传参。
翻阅了Hydra的传参规则，发现可以通过添加'++'来覆盖已有的参数。
于是我当时的sweep 配置belike：
``` yaml
command:
  - ${env}
  - python
  - ${program}
  - ${args_no_hyphens}
  - experiment=contras-cat
  - trainer=gpu
method: bayes
metric:
  goal: maximize
  name: test/F1@5
parameters:
  ++model.contrastive_weight:
    distribution: uniform
    max: 20
    min: 0.5
program: train.py
```
可以发现我是指定`args_no_hyphens`后，手动给变量名加上'++'，这样也可以潦草地实现给Hydra传参。

最近我就想着把这个思路转换成给`wandb`的pr. 

## 实现功能
- **参数追加模式**：通过`${args_append_hydra}`扩展原始配置
- **参数覆盖模式**：通过`${args_override_hydra}`修改已有参数

## 应用示例

### Case 0: 原始配置
```yaml
a: 1
b: 2
c: 3
```

### Case 1：新增实验参数
```yaml
# sweep.yaml
command:
  - ${env}
  - python
  - ${program}
  - ${args_append_hydra}
parameters:
  d:
    distribution: int_uniform
    min: 2
    max: 20
```

**配置演变**：
```yaml
# 初始配置
a: 1
b: 2
c: 3

# 追加后配置
d: [2-20]  # 新增参数区间
```

### 场景2：动态覆盖参数
```yaml
# sweep.yaml
command:
  - ${env}
  - python
  - ${program}
  - ${args_override_hydra} 
parameters:
  c:
    distribution: int_uniform
    min: 2
    max: 20
```

**配置演变**：
```yaml
# 初始配置
a: 1
b: 2

# 覆盖后配置
c: [2-20]  # 参数值被替换
```

## 核心行为对照表

| 指令                  | 作用域       | 配置影响      | 典型用例                 |
|-----------------------|-------------|---------------|--------------------------|
| `${args_append_hydra}` | 新增参数    | 配置扩展      | 探索新超参数组合         |
| `${args_override_hydra}` | 现有参数  | 值替换        | 优化已知参数的有效范围   |

## 总结
可以看到这些变更没有什么技术含量。但是这毕竟是我第一次正经Pull Request（除开改错别字xD）。还是期待 Owner 快速 Review，然后让我 merge 吧 :D。

*本文档为技术提案草案，实际功能以最终合并版本为准。欢迎在[PR讨论区](https://github.com/wandb/wandb/pull/1856)参与技术方案讨论。*
