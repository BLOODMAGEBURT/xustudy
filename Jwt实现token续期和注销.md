### Jwt实现token续期和注销

后端实现思路：

1. 在登录接口中 如果校验账号密码成功 则根据用户id和用户类型创建jwt token(有效期设置为-1，即永不过期),得到A
2. 更新登录日期(当前时间new Date()即可)（业务上可选），得到B
3. 在redis中缓存key为ACCESS_TOKEN:userId:A(加上A是为了防止用户多个客户端登录 造成token覆盖),value为B的毫秒数（转换成字符串类型），过期时间为7天（7 * 24 * 60 * 60）
4. 在登录结果中返回json格式为{"result":"success","token": A}
5. 用户在接口请求header中携带token进行登录，后端在所有接口前置拦截器进行拦截，作用是解析token 拿到userId和用户类型（用户调用业务接口只需要传token即可）， 如果解析失败（抛出SignatureException），则返回json（code = 0 ,info= Token验证不通过, errorCode = '1001'）； 此外如果解析成功，验证redis中key为ACCESS_TOKEN:userId:A 是否存在 如果不存在 则返回json（code = 0 ,info= 会话过期请重新登录, errorCode = '1002'）； 如果缓存key存在，则自动续7天超时时间（value不变），实现频繁登录用户免登陆。
6. 把userId和用户类型放入request参数中 接口方法中可以直接拿到登录用户信息
7. 如果是修改密码或退出登录 则废除access_tokens（删除key）