# 偶尔看群说话

AstrBot 群聊插件：偶尔看群、按动态帧判断要不要接话，并在不合适的时候保持沉默。

它的目标不是让 bot 对每条消息都回复，而是把群聊回复拆成两层：

1. **短上下文判定**：只看最近文本，决定要不要接，并给出一个简短的 `intent`；
2. **全量上下文生成**：判定通过后，走 AstrBot 正常会话链路，用完整人格和历史生成最终回复。

## 功能

- 每个群独立的看群周期和看群窗口
- 被 @、引用 bot、叫 bot 名字、触发词、戳一戳时立即进入看群状态
- 动态帧间隔：群聊越活跃，判定越密；连续沉默则退避
- 短判定只做决策，不写最终正文
- 全量上下文阶段可以使用 AstrBot 的图片转文字（i2t）记录
- 打分阶段忽略图片/语音，图片相关消息不会被短判定“假装看过”
- `<silent>` 等标记拦截
- 可选独立审查器
- 可选独立判定模型

## 安装

把本目录放到 AstrBot 的 `data/plugins/` 下，重启 AstrBot。

配置由 AstrBot 保存在自己的 `data/config/` 目录中，不会写进插件仓库。

## 主要配置

| 配置 | 默认 | 说明 |
|---|---|---|
| `group_policy` | `hint` | `hint` 开看群；`mention` 只有被点名才回复；`off` 完全不接管 |
| `proactive_enabled` | `true` | 是否开启主动看群判定 |
| `proactive_whitelist` | `[]` | 主动判定群白名单；留空则读取 AstrBot `active_reply.whitelist`，两者都为空则不主动判定 |
| `watch_period_seconds` | `600` | 每个群的自然开窗周期 |
| `watch_duration_seconds` | `180` | 每次自然窗口持续多久 |
| `at_boost_seconds` | `180` | 被 @/引用/叫名字/戳一戳后额外看群多久 |
| `frame_min_seconds` | `45` | 最小帧间隔 |
| `frame_base_seconds` | `90` | 普通活跃度帧间隔 |
| `frame_max_seconds` | `180` | 最大帧间隔 |
| `max_checks_per_window` | `2` | 每个窗口最多判定几次 |
| `min_proactive_gap_seconds` | `300` | 两次主动发言最小间隔 |
| `proactive_min_confidence` | `0.5` | 短判定最低置信度 |
| `banter_review_floor` | `0.3` | 低置信度玩梗仍可送审的下限 |
| `decision_provider_id` | `""` | 短判定模型；留空用当前主模型 |
| `reviewer_enabled` | `true` | 是否启用生成后审查器 |
| `reviewer_provider_id` | `qwen/qwen3.7-flash` | 审查器模型 |
| `decision_timeout_seconds` | `15` | 短判定超时 |
| `reviewer_timeout_seconds` | `5` | 审查器超时 |

## 行为说明

- 没有文本的纯图片/纯表情消息不会触发主动判定。
- 图片和语音不会进入短判定上下文；真正的回复阶段使用 AstrBot 正常链路提供的 i2t/caption 记录。
- 判定阶段返回的 `intent` 只是软提示；完整上下文发现判断不对时，可以改方向或输出 `<silent>`。
- 静默标记默认包含 `<silent>`、`[[silent]]`、`<NO_RESPONSE>`、`<NO_REPLY>`、`<NO_ANSWER>`、`<SKIP>`。
- 使用 Qwen 作为判定模型时，建议在 provider 的 `custom_extra_body` 中设置 `enable_thinking: false`，否则输出 token 和延迟会明显升高。

## 目录

```text
astrbot_plugin_occasional_chat/
├── main.py
├── metadata.yaml
├── _conf_schema.json
└── README.md
```

## License

未指定。使用或分发前请先联系作者。
