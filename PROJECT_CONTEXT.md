# PROJECT_CONTEXT.md

> File ngữ cảnh dự án cho **AUTO-EDIT-VIDEO**.  
> Mục đích: giúp người phát triển hoặc AI assistant đọc vào là hiểu ngay mục tiêu, phạm vi, kiến trúc và quy tắc triển khai của dự án mà không cần hỏi lại nhiều.

---

## 1. Tên định hướng sản phẩm

**Remotion Template Automation Editor**

Tên repo hiện tại: `AUTO-EDIT-VIDEO`.

Đây không phải là bản clone Premiere. Đây là một phần mềm biên tập video tự động hóa, có giao diện quen thuộc kiểu editor video, nhưng lõi bên trong được xây theo tư duy của Remotion:

```txt
Video = data template + React/Remotion renderer + effect preset + input batch
```

---

## 2. Mục tiêu chính

Xây dựng một phần mềm giúp người dùng:

1. Kéo video, ảnh, audio, text vào project.
2. Sắp xếp chúng thành layer trên timeline.
3. Gắn effect vào từng layer.
4. Chỉnh tham số effect trong panel effect controls.
5. Lưu bộ effect thành preset.
6. Tạo template video có placeholder như `{{title}}`, `{{image}}`, `{{video}}`, `{{price}}`.
7. Nhập dữ liệu hàng loạt từ JSON/CSV/Excel/API.
8. Áp input vào template.
9. Preview từng output.
10. Xuất hàng loạt video.

Luồng tổng quát:

```txt
Import assets
  → create layers
  → add effects
  → save presets
  → create template
  → map input data
  → preview generated video
  → batch render outputs
```

---

## 3. Điều dự án KHÔNG cố gắng làm

Dự án không hướng tới việc thay thế Premiere đầy đủ.

Không làm ở giai đoạn đầu:

- Multi-cam editing.
- Audio mixer phức tạp.
- Keyframe editor nâng cao như After Effects.
- Mask/rotoscoping nâng cao.
- Nested sequence phức tạp.
- Plugin marketplace.
- AI auto-edit phức tạp.
- Cloud rendering ngay từ đầu.
- UI quá nhiều tính năng trước khi có engine ổn định.

Ưu tiên số một là:

```txt
Một template tạo một lần
→ thay dữ liệu nhiều lần
→ xuất hàng loạt video tự động
```

---

## 4. Tư duy khác Premiere

Premiere dùng JSX/ExtendScript để điều khiển phần mềm:

```txt
Script ra lệnh cho editor:
addClip()
moveClip()
applyEffect()
export()
```

Dự án này dùng Remotion nên không đi theo hướng đó.

Tư duy đúng:

```txt
State/data điều khiển video:
project.assets
project.timeline.layers
layer.effects
template.placeholders
renderJobs
```

Tức là mọi thao tác người dùng làm trong UI sẽ biến thành dữ liệu.  
Preview và render đều đọc từ dữ liệu này.

---

## 5. Ngôn ngữ và định dạng trong dự án

### 5.1. TypeScript / TSX

Dùng cho:

- Remotion components.
- Layer renderer.
- Effect engine.
- Built-in effects.
- Batch render scripts.
- Developer plugins.

### 5.2. JSON

Dùng cho:

- Project file.
- Template file.
- Preset file.
- Input data.
- Render job data.

### 5.3. Không dùng JSX kiểu Premiere

Không dùng ExtendScript/Premiere JSX làm ngôn ngữ chính.

Có thể dùng khái niệm "script/plugin", nhưng trong dự án này script/plugin nên là:

```txt
TypeScript module
hoặc
JSON automation file
```

---

## 6. Các khái niệm cốt lõi

### Project

Toàn bộ dự án đang làm việc.

```ts
type Project = {
  id: string;
  name: string;
  assets: Asset[];
  templates: Template[];
  presets: EffectPreset[];
  activeTemplateId: string;
};
```

### Asset

File được import vào project.

```ts
type Asset = {
  id: string;
  type: 'image' | 'video' | 'audio' | 'font';
  name: string;
  src: string;
  durationInFrames?: number;
  width?: number;
  height?: number;
};
```

### Layer

Một item nằm trên timeline.

```ts
type Layer = {
  id: string;
  name: string;
  type: 'text' | 'image' | 'video' | 'audio' | 'shape';
  trackIndex: number;
  startFrame: number;
  durationInFrames: number;
  source?: string;
  text?: string;
  layout: LayerLayout;
  effects: LayerEffect[];
};
```

### LayerLayout

Thông tin vị trí, kích thước, opacity, rotation.

```ts
type LayerLayout = {
  x: number;
  y: number;
  width: number;
  height: number;
  scale: number;
  rotation: number;
  opacity: number;
};
```

### Effect

Một hiệu ứng đơn lẻ được áp vào layer.

```ts
type LayerEffect = {
  id: string;
  presetId: string;
  enabled: boolean;
  params: Record<string, unknown>;
};
```

### Preset

Một effect hoặc một bộ effect đã lưu để dùng lại.

```ts
type EffectPreset = {
  id: string;
  name: string;
  target: 'any' | 'text' | 'image' | 'video' | 'audio' | 'shape';
  effects: Array<{
    presetId: string;
    params: Record<string, unknown>;
  }>;
};
```

### Template

Timeline có thể chứa placeholder để render nhiều output.

```ts
type Template = {
  id: string;
  name: string;
  width: number;
  height: number;
  fps: number;
  durationInFrames: number;
  layers: Layer[];
  placeholders: Placeholder[];
};
```

### Placeholder

Biến sẽ được thay bằng input.

```ts
type Placeholder = {
  key: string;
  label: string;
  type: 'text' | 'image' | 'video' | 'audio' | 'number';
  required: boolean;
};
```

Ví dụ placeholder:

```txt
{{title}}
{{subtitle}}
{{image}}
{{video}}
{{price}}
{{voice}}
```

### InputRow

Một dòng dữ liệu dùng để sinh một video output.

```ts
type InputRow = {
  id: string;
  outputName: string;
  values: Record<string, string | number | boolean | null>;
};
```

### RenderJob

Một job render cụ thể.

```ts
type RenderJob = {
  id: string;
  templateId: string;
  inputRowId: string;
  outputName: string;
  status: 'queued' | 'rendering' | 'done' | 'failed';
};
```

---

## 7. UI mục tiêu giai đoạn đầu

Giao diện nên giống editor video đơn giản:

```txt
┌────────────────────────────────────────────────────┐
│ Project / Assets       │ Preview                  │
│                        │ Remotion Player          │
├────────────────────────┼──────────────────────────┤
│ Effects                │ Effect Controls          │
│ Built-in + Presets     │ Params of selected layer │
├────────────────────────┴──────────────────────────┤
│ Timeline                                          │
│ Tracks + layers + duration                        │
└────────────────────────────────────────────────────┘
```

### Project / Assets

- Hiển thị ảnh, video, audio, font đã import.
- Kéo asset vào timeline để tạo layer.
- Giai đoạn đầu có thể dùng file từ `public/`.

### Preview

- Dùng Remotion Player để xem template hiện tại.
- Preview nhận `projectState`, `template`, `selectedInputRow` làm props.

### Timeline

- Hiển thị track và layer.
- Cho phép sửa:
  - `startFrame`
  - `durationInFrames`
  - `trackIndex`
  - thứ tự layer
  - selected layer

### Effects

- Danh sách effect có sẵn.
- Danh sách preset người dùng đã lưu.

### Effect Controls

- Khi chọn layer, hiển thị các effect đang gắn vào layer.
- Cho phép chỉnh params.
- Cho phép bật/tắt effect.
- Cho phép lưu bộ effect hiện tại thành preset.

---

## 8. Effect engine

Effect engine phải tách khỏi UI.

UI chỉ làm nhiệm vụ:

```txt
Chọn effect
→ chỉnh params
→ lưu vào layer.effects
```

Renderer làm nhiệm vụ:

```txt
Đọc layer.effects
→ tìm effect implementation
→ tính style theo frame
→ render layer
```

Mẫu API nội bộ:

```ts
type EffectContext = {
  frame: number;
  localFrame: number;
  fps: number;
  durationInFrames: number;
  layer: Layer;
};

type EffectImplementation = {
  id: string;
  name: string;
  target: 'any' | 'text' | 'image' | 'video' | 'audio' | 'shape';
  controls: EffectControlSchema;
  apply: (
    context: EffectContext,
    params: Record<string, unknown>
  ) => React.CSSProperties;
};
```

Effect nên trả về style hoặc data transform.  
Không nên để effect tự render toàn bộ layer, trừ các effect đặc biệt.

Built-in effects giai đoạn đầu:

```txt
fade-in
fade-out
slide-in-left
slide-in-right
slide-in-up
slide-in-down
zoom-in
zoom-out
pop
blur-in
shake
typewriter
```

---

## 9. Preset system

Preset người dùng lưu nên là JSON, không phải code.

Ví dụ file `.rempreset.json`:

```json
{
  "id": "title-enter-left-soft",
  "name": "Title Enter Left Soft",
  "target": "text",
  "effects": [
    {
      "presetId": "fade-in",
      "params": {
        "duration": 15
      }
    },
    {
      "presetId": "slide-in-left",
      "params": {
        "duration": 20,
        "distance": 240
      }
    }
  ]
}
```

Preset chỉ mô tả:

```txt
Dùng effect nào
+ params gì
+ áp dụng cho loại layer nào
```

Preset không chứa function JavaScript để tránh phức tạp, khó validate, khó bảo mật.

---

## 10. Plugin system

Plugin dành cho developer, không phải người dùng phổ thông.

Plugin nên là TypeScript module đã được compile cùng app.

Ví dụ ý tưởng:

```ts
export const myCustomEffect: EffectImplementation = {
  id: 'my-custom-effect',
  name: 'My Custom Effect',
  target: 'any',
  controls: {},
  apply: (context, params) => {
    return {};
  },
};
```

Quy tắc:

- Built-in effect nằm trong `src/effects/`.
- User preset nằm trong JSON.
- Developer plugin có thể nằm trong `src/plugins/`.
- Không chạy code plugin không tin cậy từ bên ngoài trong giai đoạn đầu.
- Nếu cần import plugin ngoài trong tương lai, phải có sandbox hoặc cơ chế kiểm soát riêng.

---

## 11. Template file

Template nên lưu bằng JSON.

Ví dụ `.remtemplate.json`:

```json
{
  "id": "product-video-basic",
  "name": "Product Video Basic",
  "width": 1920,
  "height": 1080,
  "fps": 30,
  "durationInFrames": 120,
  "layers": [
    {
      "id": "background-video",
      "name": "Background Video",
      "type": "video",
      "trackIndex": 0,
      "startFrame": 0,
      "durationInFrames": 120,
      "source": "{{video}}",
      "layout": {
        "x": 0,
        "y": 0,
        "width": 1920,
        "height": 1080,
        "scale": 1,
        "rotation": 0,
        "opacity": 1
      },
      "effects": [
        {
          "id": "effect-1",
          "presetId": "zoom-in",
          "enabled": true,
          "params": {
            "from": 1,
            "to": 1.08,
            "duration": 120
          }
        }
      ]
    },
    {
      "id": "title",
      "name": "Title",
      "type": "text",
      "trackIndex": 1,
      "startFrame": 10,
      "durationInFrames": 90,
      "text": "{{title}}",
      "layout": {
        "x": 120,
        "y": 760,
        "width": 1400,
        "height": 160,
        "scale": 1,
        "rotation": 0,
        "opacity": 1
      },
      "effects": [
        {
          "id": "effect-2",
          "presetId": "slide-in-left",
          "enabled": true,
          "params": {
            "duration": 18,
            "distance": 300
          }
        }
      ]
    }
  ],
  "placeholders": [
    {
      "key": "video",
      "label": "Video nền",
      "type": "video",
      "required": true
    },
    {
      "key": "title",
      "label": "Tiêu đề",
      "type": "text",
      "required": true
    }
  ]
}
```

---

## 12. Project file

Project nên lưu toàn bộ trạng thái editor.

Ví dụ `.remproject.json`:

```json
{
  "id": "auto-edit-video-demo",
  "name": "Auto Edit Video Demo",
  "activeTemplateId": "product-video-basic",
  "assets": [],
  "templates": [],
  "presets": []
}
```

Giai đoạn đầu có thể lưu vào local file hoặc localStorage.  
Sau này có thể lưu bằng database.

---

## 13. Input và batch rendering

Input có thể là JSON/CSV/Excel/API, nhưng nội bộ nên normalize về một dạng duy nhất:

```ts
type BatchInput = {
  rows: InputRow[];
};
```

Ví dụ JSON:

```json
{
  "rows": [
    {
      "id": "row-1",
      "outputName": "video-product-1.mp4",
      "values": {
        "title": "Sản phẩm A",
        "video": "videos/product-a.mp4",
        "price": "199.000đ"
      }
    },
    {
      "id": "row-2",
      "outputName": "video-product-2.mp4",
      "values": {
        "title": "Sản phẩm B",
        "video": "videos/product-b.mp4",
        "price": "299.000đ"
      }
    }
  ]
}
```

Render batch:

```txt
For each row:
  resolve placeholders
  build inputProps
  select composition
  render media
  save mp4
```

---

## 14. Remotion rules cho dự án

### Composition mặc định

Nếu chưa có yêu cầu khác, dùng:

```txt
composition id: MyComp
width: 1920
height: 1080
fps: 30
durationInFrames: 120
```

### Entry files

Cấu trúc Remotion cơ bản:

```txt
src/index.ts
src/Root.tsx
src/Composition.tsx
```

### Preview

Preview trong editor nên dùng `@remotion/player`.

### Render

Render bằng CLI:

```bash
npx remotion render MyComp
```

Render still:

```bash
npx remotion still MyComp
```

Render hàng loạt bằng Node API nên dùng `@remotion/renderer`.

### Asset

Asset local để trong `public/`.

Dùng:

```ts
staticFile('path/to/file.png')
```

Không hardcode đường dẫn public kiểu `'/file.png'` trong component render.

### Deterministic rendering

Không dùng:

```ts
Math.random()
```

Nếu cần random, dùng:

```ts
random('static-seed')
```

### Animation

Ưu tiên Remotion APIs:

```txt
useCurrentFrame()
useVideoConfig()
interpolate(..., { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' })
spring()
Sequence
Series
AbsoluteFill
```

### Media

Dùng đúng component:

```txt
Video / Audio từ @remotion/media
Img / staticFile từ remotion
Gif từ @remotion/gif nếu cần animated GIF
```

---

## 15. Kiến trúc thư mục đề xuất

```txt
src/
  index.ts
  Root.tsx
  Composition.tsx

  app/
    EditorApp.tsx
    panels/
      PreviewPanel.tsx
      TimelinePanel.tsx
      ProjectPanel.tsx
      EffectsPanel.tsx
      EffectControlsPanel.tsx
      InputPanel.tsx
      RenderPanel.tsx

  engine/
    types.ts
    TemplateRenderer.tsx
    LayerRenderer.tsx
    applyEffects.ts
    resolveInputTokens.ts
    buildRenderProps.ts
    validateProject.ts

  effects/
    fadeIn.ts
    fadeOut.ts
    slideIn.ts
    zoomIn.ts
    blurIn.ts
    shake.ts
    typewriter.ts
    index.ts

  presets/
    defaultPresets.ts

  templates/
    sampleTemplate.ts

  plugins/
    README.md

  render/
    renderBatch.ts

public/
  images/
  videos/
  audios/
  fonts/

data/
  sample-input.json
  sample-project.remproject.json
  sample-template.remtemplate.json
```

---

## 16. Trạng thái ưu tiên hiện tại

Hiện dự án nên ưu tiên xây nền móng, chưa ưu tiên UI đẹp.

Thứ tự làm:

1. Chuẩn hóa Remotion composition.
2. Tạo `engine/types.ts`.
3. Tạo sample template.
4. Tạo `TemplateRenderer`.
5. Tạo `LayerRenderer`.
6. Tạo effect registry.
7. Làm 3 effect đầu: `fade-in`, `slide-in-left`, `zoom-in`.
8. Render được template từ JSON.
9. Thêm Remotion Player preview.
10. Làm timeline UI đơn giản.
11. Làm effect controls.
12. Lưu preset JSON.
13. Import input JSON/CSV.
14. Render batch.

---

## 17. MVP v0.1

MVP v0.1 được coi là thành công nếu làm được:

```txt
Một template JSON
+ một input JSON có nhiều rows
+ một effect preset JSON
→ preview được từng row
→ render được nhiều mp4 output
```

UI chỉ cần đơn giản:

- Preview.
- Timeline thô.
- Layer list.
- Effect list.
- Effect controls.
- Import input JSON.
- Render button.

---

## 18. Quy tắc khi AI assistant làm việc với repo này

Khi một AI assistant đọc dự án này, phải hiểu rằng:

1. Đây là **Remotion Template Automation Editor**, không phải clone Premiere.
2. Không nên hỏi lại mục tiêu cơ bản; mục tiêu đã được ghi trong file này.
3. Mọi tính năng nên phục vụ batch video automation.
4. Không hardcode một video cụ thể nếu có thể biểu diễn bằng template/layer/effect data.
5. Preset người dùng nên lưu bằng JSON.
6. Plugin developer nên viết bằng TypeScript.
7. UI chỉ chỉnh state; renderer mới là nơi tạo video.
8. Không dùng Premiere JSX/ExtendScript.
9. Không dùng `Math.random()` trong render code.
10. Luôn ưu tiên code Remotion/React/TypeScript hợp lệ và deterministic.
11. Khi viết hoặc sửa Remotion code, kiểm tra tài liệu Remotion chính thức hiện tại trước.
12. Nếu thiếu chi tiết nhỏ, tự chọn default hợp lý:
    - `MyComp`
    - `1920x1080`
    - `30fps`
    - `120 frames`
    - `defaultProps` khớp với props của component.

---

## 19. Câu nói tóm tắt dự án

```txt
AUTO-EDIT-VIDEO là một editor video tự động hóa dựa trên Remotion,
cho phép tạo template bằng layer/effect/preset,
sau đó thay input hàng loạt để xuất nhiều video.
```

---

## 20. Ghi chú phát triển dài hạn

Sau MVP, có thể mở rộng:

- Desktop app bằng Electron hoặc Tauri.
- Local render queue.
- Import CSV/Excel nâng cao.
- Cloud render.
- Template marketplace.
- AI tạo caption/script.
- Auto resize nhiều tỉ lệ: 16:9, 9:16, 1:1.
- Render nhiều format.
- Plugin system an toàn hơn.
- Versioning cho template/preset.

Nhưng tất cả phần mở rộng này phải dựa trên nền móng:

```txt
Project state
→ Template
→ Layers
→ Effects
→ Presets
→ Input
→ Render jobs
```
