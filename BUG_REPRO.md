# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

管理员给一个已被删除的客户 ID 录入宠物时，接口返回笼统的 500，后台看不出是归属对象不存在。请修复错误传播，让这类外键冲突返回可识别的 409，并确认失败事务不会留下没有主人的宠物记录；不得修改现有测试，也不能跳过回滚验证。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/pet-foster-task-11
- 仓库地址：https://github.com/zhanglei10281852-gif/pet-foster-task-11.git
- parent SHA：5dc0c1a3ff5cd4de9b7b84131d4e302f2d3a643a

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/pet-foster-task-11.git bug-repro
cd bug-repro
git checkout --detach 5dc0c1a3ff5cd4de9b7b84131d4e302f2d3a643a
go test ./internal/pet -run ^TestAnnotationMissingPetOwnerConflict$ -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pet -run ^TestAnnotationMissingPetOwnerConflict$ -count=1
2026/08/20 17:08:44 INFO pet http request request_id=req_60de9a6c0080924ebdad9b6d method=POST path=/api/pet/add status=500 duration_ms=0
--- FAIL: TestAnnotationMissingPetOwnerConflict (0.17s)
    annotation_pet_behavior_test.go:213: status=500 body={"code":500,"data":null,"message":"服务内部错误","requestId":"req_60de9a6c0080924ebdad9b6d"}
FAIL
FAIL	github.com/zhanglei10281852-gif/pet-foster-go/internal/pet	0.176s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pet -run ^TestAnnotationMissingPetOwnerConflict$ -count=1
2026/08/20 17:18:29 INFO pet http request request_id=req_6dc7a6e04e46e9db97998eec method=POST path=/api/pet/add status=500 duration_ms=25
--- FAIL: TestAnnotationMissingPetOwnerConflict (0.65s)
    annotation_pet_behavior_test.go:213: status=500 body={"code":500,"data":null,"message":"服务内部错误","requestId":"req_6dc7a6e04e46e9db97998eec"}
FAIL
FAIL	github.com/zhanglei10281852-gif/pet-foster-go/internal/pet	0.864s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

为不存在的客户 ID 创建宠物必须返回可识别的 409，失败事务中不得留下宠物或其他关联记录；有效客户的创建流程仍可成功。TestAnnotationMissingPetOwnerConflict 应完成修复前失败、修复后通过的验证，相关包和全量回归无异常；不得改测试、跳过回滚检查或吞掉外键冲突。
