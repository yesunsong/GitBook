# Pass 可选配置参数

默认值为加粗项，所有参数不区分大小写。

|                 Name                  |                           Options                            |
| :-----------------------------------: | :----------------------------------------------------------: |
|                switch                 | ***undefined**, could be any valid macro name that's not defined in the shader |
|               priority                | **default**(128), could be any number between max(255) and min(0) |
|                 stage                 | **default**, could be the name of any registered stage in your runtime pipeline |
|              properties               |                  *see the following section                  |
|              migrations               |                  *see the following section                  |
|               primitive               | point_list, line_list, line_strip, line_loop, **triangle_list**, triangle_strip, triangle_fan, line_list_adjacency, line_strip_adjacency, triangle_list_adjacency, triangle_strip_adjacency, triangle_patch_adjacency, quad_patch_list, iso_line_list |
|               dynamics                | **[]**, an array containing any of the following: viewport, scissor, line_width, depth_bias, blend_constants, depth_bounds, stencil_write_mask, stencil_compare_mask |
|       rasterizerState. cullMode       |                    front, **back**, none                     |
|     depthStencilState. depthTest      |                       **true**, false                        |
|     depthStencilState. depthWrite     |                       **true**, false                        |
|     depthStencilState. depthFunc      | never, **less**, equal, less_equal, greater, not_equal, greater_equal, always |
|     blendState.targets[i]. blend      |                       true, **false**                        |
|    blendState.targets[i]. blendEq     |                    **add**, sub, rev_sub                     |
|    blendState.targets[i]. blendSrc    | **one**, zero, src_alpha_saturate, src_alpha, one_minus_src_alpha, dst_alpha, one_minus_dst_alpha, src_color, one_minus_src_color, dst_color, one_minus_dst_color, constant_color, one_minus_constant_color, constant_alpha, one_minus_constant_alpha |
|    blendState.targets[i]. blendDst    | one, **zero**, src_alpha_saturate, src_alpha, one_minus_src_alpha, dst_alpha, one_minus_dst_alpha, src_color, one_minus_src_color, dst_color, one_minus_dst_color, constant_color, one_minus_constant_color, constant_alpha, one_minus_constant_alpha |
| blendState.targets[i]. blendSrcAlpha  | **one**, zero, src_alpha_saturate, src_alpha, one_minus_src_alpha, dst_alpha, one_minus_dst_alpha, src_color, one_minus_src_color, dst_color, one_minus_dst_color, constant_color, one_minus_constant_color, constant_alpha, one_minus_constant_alpha |
| blendState.targets[i]. blendDstAlpha  | one, **zero**, src_alpha_saturate, src_alpha, one_minus_src_alpha, dst_alpha, one_minus_dst_alpha, src_color, one_minus_src_color, dst_color, one_minus_dst_color, constant_color, one_minus_constant_color, constant_alpha, one_minus_constant_alpha |
|  blendState.targets[i]. blendAlphaEq  |                    **add**, sub, rev_sub                     |
| blendState.targets[i]. blendColorMask | **all**, none, r, g, b, a, rg, rb, ra, gb, ga, ba, rgb, rga, rba, gba |
|        blendState. blendColor         |                  **0** or **[0, 0, 0, 0]**                   |
|    depthStencilState. stencilTest     |                       true, **false**                        |
|    depthStencilState. stencilFunc     | never, less, equal, less_equal, greater, not_equal, greater_equal, **always** |
|  depthStencilState. stencilReadMask   |              **0xffffffff** or **[1, 1, 1, 1]**              |
|  depthStencilState. stencilWriteMask  |              **0xffffffff** or **[1, 1, 1, 1]**              |
|   depthStencilState. stencilFailOp    | **keep**, zero, replace, incr, incr_wrap, decr, decr_wrap, invert |
|   depthStencilState. stencilZFailOp   | **keep**, zero, replace, incr, incr_wrap, decr, decr_wrap, invert |
|   depthStencilState. stencilPassOp    | **keep**, zero, replace, incr, incr_wrap, decr, decr_wrap, invert |
|     depthStencilState. stencilRef     |                  **1** or **[0, 0, 0, 1]**                   |
| depthStencilState. stencil*Front/Back |      *\*set above stencil properties for specific side*      |

## Switch

指定这个 pass 的执行依赖于哪个 define，它不应与使用到的 shader 中定义的任何 define 重名。
这个字段默认是不存在的，意味着这个 pass 是无条件执行的。

## Priority

指定这个 pass 的渲染优先级，数值越小越优先渲染；default 代表默认优先级 (128)，min 代表最小（0），max 代表最大（255），可结合四则运算符指定相对值。

## Stage

指定这个 pass 归属于管线的哪个 stage，对 forward 管线，只有 default 一个 stage。

## Properties

properties 存储着这个 Pass 有哪些可定制的参数需要在 Inspector 上显示，
这些参数可以是 shader 中的某个 uniform 的完整映射，也可以是具体某个分量的映射 (使用 target 参数)：

```yaml
albedo: { value: [1, 1, 1, 1] } # uniform vec4 albedo
roughness: { value: 0.8, target: pbrParams.g } # uniform vec4 pbrParams
offset: { value: [0, 0], target: tilingOffset.zw } # uniform vec4 tilingOffset
# say there is another uniform, vec4 emissive, that doesn't appear here
# so it will be assigned a default value of [0, 0, 0, 0] and will not appear in the inspector
```

运行时可以这样使用：

```js
// as long as it is a real uniform
// it doesn't matter whether it is specified in the property list or not
mat.setProperty('emissive', cc.Color.GREY); // this works
mat.setProperty('albedo', cc.Color.RED); // directly set uniform
mat.setProperty('roughness', 0.2); // set certain component
const h = mat.passes[0].getHandle('offset'); // or just take the handle,
mat.passes[0].setUniform(h, new Vec2(0.5, 0.5)); // and use Pass.setUniform interface instead
```

未指定的 uniform 将由引擎在运行时根据自动分析出的数据类型给予[默认初值](https://docs.cocos.com/creator3d/manual/zh/material-system/pass-parameter-list.html#default-values)。

为方便声明各 property 子属性，可以直接在 properties 内声明 `__metadata__` 项，所有 property 都会继承它声明的内容，如：

```yaml
properties:
  __metadata__: { editor: { visible: false } }
  a: { value: [1, 1, 0, 0] }
  b: { editor: { type: color } }
  c: { editor: { visible: true } }
```

这样 uniform a 和 b 已声明的各项参数都不受影响，但全部不会显示在 inspector 上（visible 为 false），而 uniform c 还会正常显示。

## Migrations

一般来说使用材质资源时希望底层的 effect 接口能始终向前兼容，但依然有时面对新的需求最好的解决方案是含有一定 breaking change 的，
这时为了保持项目中已有的材质资源数据不受影响，或至少能够更平滑的升级，可以使用 effect 的迁移系统，
在 effect 导入成功后会 **立即更新工程内所有** 依赖于此 effect 的材质资源，
对每个材质资源，尝试寻找所有指定旧参数数据（包括 property 和宏定义两类），复制或重组到新属性名下。
这个过程不会自动删除旧数据，但是会将旧属性的 editor.deprecated 标为 true（如果新数据在 inspector 上可见的话）。
如果一个材质资源内既有旧数据，又有新数据，则不会做任何迁移（强制更新模式除外）。

对于一个现有 effect，声明如下迁移字段：

```yaml
migrations:
  # macros: # macros follows the same rule as properties, without the component-wise features
  #   USE_MIAN_TEXTURE: { formerlySerializedAs: USE_MAIN_TEXTURE }
  properties:
    newFloat: { formerlySerializedAs: oldVec4.w }
```

对于一个依赖于这个 effect，并在对应 pass 持有这样的属性的材质：

```json
{
  "oldVec4": {
    "__type__": "cc.Vec4",
    "x": 1,
    "y": 1,
    "z": 1,
    "w": 0.5
  }
}
```

在 effect 重新导入后，这些数据会被立即转换成：

```json
{
  "oldVec4": {
    "__type__": "cc.Vec4",
    "x": 1,
    "y": 1,
    "z": 1,
    "w": 0.5
  },
  "newFloat": 0.5
}
```

在编辑器内重新编辑并保存这个材质资源后会变成（假设 effect 和 property 数据本身并没有改变）：

```json
{
  "newFloat": 0.5
}
```

注意这里的通道指令只是简单的取 `w` 分量，事实上还可以做任意的 shuffle：

```yaml
    newColor: { formerlySerializedAs: someOldColor.yxx }
```

甚至基于某个宏定义：

```yaml
    occlusion: { formerlySerializedAs: pbrParams.<OCCLUSION_CHANNEL|z> }
```

这里声明了新的 occlusion 属性会从旧的 `pbrParams` 中获取，而具体的分量取决于 OCCLUSION_CHANNEL 宏定义，且如材质资源中未定义此宏，默认取 `z` 通道。
但如果某个材质在迁移升级前就已经存着 `newFloat` 字段的数据，则不会对其做任何修改，除非指定为强制更新模式：

```yaml
    newFloat: { formerlySerializedAs: oldVec4.w! }
```

这会强制更新所有材质的属性，无论这个操作是否会覆盖数据。
注意强制更新操作会在编辑器的每次资源事件中都执行（几乎对应每一次鼠标点击，相对高频），
因此只是一个快速测试和调试的手段，一定不要将处于强制更新模式的 effect 提交版本控制。

## Property Param List

同样地，任何可配置字段如与默认值相同都可以省掉。

|         Param          |                           Options                            |
| :--------------------: | :----------------------------------------------------------: |
|         target         | **undefined**, (any valid uniform components, no random swizzle) |
|         value          |                   *\*see the next section*                   |
|   sampler. minFilter   |             none, point, **linear**, anisotropic             |
|   sampler. magFilter   |             none, point, **linear**, anisotropic             |
|   sampler. mipFilter   |             **none**, point, linear, anisotropic             |
|   sampler. addressU    |               **wrap**, mirror, clamp, border                |
|   sampler. addressV    |               **wrap**, mirror, clamp, border                |
|   sampler. addressW    |               **wrap**, mirror, clamp, border                |
| sampler. maxAnisotropy |                            **16**                            |
|    sampler. cmpFunc    | **never**, less, equal, less_equal, greater, not_equal, greater_equal, always |
|  sampler. borderColor  |                       **[0, 0, 0, 0]**                       |
|    sampler. minLOD     |                            **0**                             |
|    sampler. maxLOD     | **0**, *\*remember to override this when enabling mip filter* |
|  sampler. mipLODBias   |                            **0**                             |
|         editor         |                   *\*see the next section*                   |
|  editor. displayName   |               (any string), ***property name**               |
|      editor. type      |                      **vector**, color                       |
|    editor. visible     |                       **true**, false                        |
|    editor. tooltip     |               (any string), ***property name**               |
|     editor. range      |             **undefined**, [ min, max, [step] ]              |
|   editor. deprecated   | true, **false**, *\*for any material using this effect, delete the existing data for this property after next saving* |

## Default Values

|    Type     |         Default Value / Options          |
| :---------: | :--------------------------------------: |
|     int     |                    0                     |
|    ivec2    |                  [0, 0]                  |
|    ivec3    |                [0, 0, 0]                 |
|    ivec4    |               [0, 0, 0, 0]               |
|    float    |                    0                     |
|    vec2     |                  [0, 0]                  |
|    vec3     |                [0, 0, 0]                 |
|    vec4     |               [0, 0, 0, 0]               |
|  sampler2D  | black, grey, white, normal, **default**  |
| samplerCube | black-cube, white-cube, **default-cube** |

对于 defines：
boolean 类型默认值为 false。
number 类型默认值为 0，默认取值范围 [0, 3]。
string 类型默认值为 options 数组第一个元素。