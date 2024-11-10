# 作业链接
https://u.geekbang.org/lesson/656?article=809465
# 作业要求
- 扩展本指南的 Reflection Agent，使其能够完成更通用的生成任务，包括但不限于代码、报告等；
- 使用扩展后的 Reflection Agent 生成代码，实现在 GitHubSentinel 上新增一个信息渠道。

# 作业解析
- 让其能完成通用任务的核心是让其具备一个决策大脑. 而已有的代码, 报告等都是属于这个决策大脑的可用工具.
- 追加决策大脑中心. 通过决策中心给与的数据来判断需要生成的内容.
```python
# 追加其他模型-反思模型
llm = ChatOpenAI(model="gpt-4o-mini", base_url=base_url, api_key=api_key)
# 写作专家
writer_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a writing assistant tasked with creating well-crafted, coherent, and engaging articles based on the user's request."
            " Focus on clarity, structure, and quality to produce the best possible piece of writing."
            " If the user provides feedback or suggestions, revise and improve the writing to better align with their expectations.",
        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)
writer = writer_prompt | llm

# 反思模型
reflection_prompt = ChatPromptTemplate.from_messages(
    [
        (
            "system",
            "You are a teacher grading an article submission. writer critique and recommendations for the user's submission."
            " Provide detailed recommendations, including requests for length, depth, style, etc.",

        ),
        MessagesPlaceholder(variable_name="messages"),
    ]
)

reflect = reflection_prompt | llm

# 包装成为tools


# 将上述的模型添加到工具列表中
# 申请一个携带反思模型的写作专家
from langchain_core.tools import tool

@tool
def write_worker(
    topic: Annotated[str, "一个优秀的文档生成模型, 这里传入的是需要生成的话题topic"],
):
    """该模型会根据用户的结果多次反思并将最终的数据结果返回."""
    try:

        reflection = ""

        write_resp = writer.invoke({"messages": [topic]})
        reflect_resp = reflect.invoke({"messages": [topic, HumanMessage(content=write_resp.content)]})

    except BaseException as e:
        return f"Failed to execute. Error: {str(e)}"


    return f"Successfully executed:\n```\n{reflect_resp.content}\n```\n"


    # result_str = f"Successfully executed:\n```python\n{code}\n```\nStdout: {result}"

    # return (
    #     result_str + "\n\nIf you have completed all tasks, respond with FINAL ANSWER."
    # )
```
# 在multi_agent_collaboration_updated.ipynb中调用
```python

from langgraph.prebuilt import ToolNode

# 定义工具列表，包括 Tavily 搜索工具和 Python REPL 工具
tools = [tavily_tool, python_repl, write_worker]

# 创建工具节点，负责工具的调用
tool_node = ToolNode(tools)

```


