<%*
const kindOptions = "WEB / MISC / PWN / RE / CRYPTO / MOBILE / BLOCKCHAIN";
const kind = await tp.system.prompt("请选择题型：\n" + kindOptions);
const platform = await tp.system.prompt("请输入平台（如：BUUCTF, XCTF）");
const folder = tp.file.folder(true).split("/").pop();
await tp.file.rename(`Write-up：${folder}`);
-%>

#wp #<% kind.toLowerCase() %>

【[[<% kind %>]]】<% folder %> - [[<% platform %>]]

创建时间：<% tp.date.now("YYYY-MM-DD HH:mm") %>

### 解题过程