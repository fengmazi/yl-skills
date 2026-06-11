# 多行文本原样显示（textarea 内容保留格式）

## 场景

展示多行文本框（textarea）输入的内容时（如审批意见、初步结论等），后端存的是带 `\n` 的原始文本，需要"填什么样，显示什么样"。

## 解决方案

```css
.summary-content {
  white-space: pre-wrap;    /* 保留换行和空格，超出自动换行 */
  word-wrap: break-word;    /* 长单词/URL 在边界断行 */
  overflow-wrap: break-word;
}
```

## 关键属性

`white-space: pre-wrap` — 保留 `<pre>` 的空白语义（换行、连续空格不折叠），同时允许超出容器宽度时自动换行（区别于 `pre` 会在溢出时出现横向滚动条）。

## 适用场景

打印模板、详情弹窗、审批意见展示等需要原样呈现用户输入格式的地方。
