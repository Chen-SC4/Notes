首先，下载 Claude Code Router 之前，需要确保电脑中安装有 `NodeJS`，版本越新越好，安装可以参考[Nvm 安装](../../前端/Nvm%20安装.md)。

来到 `Claude Code Router` [官网](https://github.com/musistudio/claude-code-router)，根据官网的教程，首先需要确保电脑中安装有 `Claude Code`。

```shell
npm install -g @anthropic-ai/claude-code
```

然后就可以安装 `Claude Code Router` 了。

```shell
npm install -g @musistudio/claude-code-router
```

###### 配置文件

`Github` 项目下面有官方推荐的配置文件写法，直接照着写即可。

```json
{
    "Providers": [
        {
            "name": "modelscope",
            "api_base_url": "https://api-inference.modelscope.cn/v1/chat/completions",
            "api_key": "ms-b068492f-0b16-41ba-81b9-6e4bbai3k4m2",
            "models": [
                "Qwen/Qwen3-Coder-480B-A35B-Instruct",
                "Qwen/Qwen3-235B-A22B-Thinking-2507"
            ],
            "transformer": {
                "use": [
                    [
                        "maxtoken",
                        {
                            "max_tokens": 65536
                        }
                    ],
                    "enhancetool"
                ],
                "Qwen/Qwen3-235B-A22B-Thinking-2507": {
                    "use": [
                        "reasoning"
                    ]
                }
            }
        }
    ],
    "Router": {
        "default": "modelscope,Qwen/Qwen3-Coder-480B-A35B-Instruct"
    }
}
```

如果需要图形化的配置界面，可以输入指令 `ccr ui` 开启一个网页来进行配置。

###### 启动命令

配置完成之后，通过 `ccr code` 命令启动 `Claude Code`。