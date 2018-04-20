# 单元测试

文档维护 | 辰枫
---|---
更新日期 | 2017-11-24
文档版本 | v1.0

## 第三方测试模块

**Testify**
* 文档：https://godoc.org/github.com/stretchr/testify
* 获取：go get github.com/stretchr/testify


```

package yours

import (
  "testing"
  "github.com/stretchr/testify/assert"
)

func TestSomething(t *testing.T) {

  // assert equality
  assert.Equal(t, 123, 123, "they should be equal")

  // assert inequality
  assert.NotEqual(t, 123, 456, "they should not be equal")

  // assert for nil (good for errors)
  assert.Nil(t, object)

  // assert for not nil (good when you expect something)
  if assert.NotNil(t, object) {

    // now we know that object isn't nil, we are safe to make
    // further assertions without causing any errors
    assert.Equal(t, "Something", object.Value)

  }

}

```