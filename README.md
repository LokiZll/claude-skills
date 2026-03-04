# Claude Skills

Claude Code 的自定义 skills 集合。

## 安装

```bash
cp -r skills/* ~/.claude/skills/
```

## Skills

| Skill | 描述 |
|-------|------|
| `design` | 设计与规划。在编码前完成需求探索、方案研究、设计验证和实施计划 |
| `tdd` | 测试驱动开发。红-绿-重构循环，涵盖单元测试、集成测试和 E2E 测试 |
| `refactor-cleaner` | 死代码清理。识别和移除未使用的代码、重复和依赖 |

## 验证

```bash
# 确认 skills 已安装
ls ~/.claude/skills/*/SKILL.md
```

## 测试

| Skill | 测试方法 |
|-------|---------|
| `design` | 说"帮我实现一个用户登录功能"，观察是否先研究、提问、设计 |
| `tdd` | 说"用 TDD 写一个计算折扣的函数"，观察是否先写测试再实现 |
| `refactor-cleaner` | 说"帮我清理死代码"，观察是否运行检测工具、分类风险 |
