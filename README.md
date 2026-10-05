# Household Expiry & Lifecycle Management (Expiry)

[![Status: Architecture & MVP Design](https://img.shields.io/badge/Status-Architecture_%26_MVP_Design-blue?style=flat-square)](https://github.com/tducn110/expiry)
[![Evidence-Based Research](https://img.shields.io/badge/Methodology-Evidence--Based_Research-success?style=flat-square)](https://github.com/tducn110/expiry)
[![Domain: Cross-Household Lifecycle](https://img.shields.io/badge/Domain-Food_%7C_Cosmetics_%7C_Medicine-orange?style=flat-square)](https://github.com/tducn110/expiry)
[![License: MIT](https://img.shields.io/badge/License-MIT-lightgrey?style=flat-square)](LICENSE)

> **"A household lifecycle assistant that keeps important products visible and tells you what needs attention next."**  
> Chuyển đổi tư duy quản lý đồ dùng gia đình từ mô hình ngày cứng nhắc (*Date-Centric*) sang mô hình vòng đời hành vi người dùng (*User-Centric & Action-Oriented Lifecycle*).

---

## 1. TỔNG QUAN BÀI TOÁN & TRIẾT LÝ HỆ THỐNG

### 1.1. Tuyên bố bài toán (Problem Statement)
Đa số các ứng dụng quản lý hạn dùng trên thị trường hiện nay xuất phát từ giả định ngây thơ:  
> *"Người dùng hay quên ngày hết hạn, vì vậy chỉ cần làm một app lưu ngày và gửi thông báo nhắc nhở."*

Nghiên cứu thực nghiệm hành vi người tiêu dùng (TotalCtrl JMIR 2022, CozZo Horizon Project, Wang et al. 2025, Kelly et al. 2018) chỉ ra rằng bài toán thực tế phức tạp hơn rất nhiều:

1. **Lệch pha kiểm kê thực tế và dữ liệu số (Physical-Digital State Drift):**  
   Tủ lạnh và kệ đồ vật lý thay đổi liên tục (mua thêm, mở nắp, nấu ăn, chuyển ngăn đông, vứt bỏ). Trong khi đó, ứng dụng chỉ cập nhật khi người dùng tự tay nhập. Gánh nặng nhập liệu (*Maintenance Friction*) khiến dữ liệu trên app nhanh chóng bị "lạc hậu" (stale). Khi thông báo gửi về không còn đúng với thực tế, người dùng mất niềm tin và bỏ ứng dụng.
2. **Thất lạc chú ý và tầm nhìn (Visibility & Attention Collapse):**  
   Đồ dùng nằm khuất phía sau tủ hoặc ngăn kéo sẽ biến mất khỏi nhận thức. Quên đồ không đơn thuần là quên thời gian, mà là **mất khả năng chú ý đúng lúc**.
3. **Hiểu sai ngữ nghĩa nhãn hạn dùng (Date Semantics Confusion):**  
   Theo FDA, sự nhầm lẫn giữa nhãn chất lượng (*Best Before*) và nhãn an toàn (*Use By*) gây ra xấp xỉ **20% lượng rác thải thực phẩm hộ gia đình**. Đối với mỹ phẩm, hạn dùng thực tế bắt đầu đếm từ ngày mở nắp (*PAO - Period After Opening*); đối với thuốc, việc lưu trữ thuốc không còn nhu cầu sử dụng tiềm ẩn rủi ro y tế nghiêm trọng.
4. **Khoảng cách giữa ý định và hành vi (Intention-Action Gap):**  
   Biết một món đồ sắp hết hạn không đồng nghĩa với việc biết cần làm gì. Thông báo chỉ có giá trị khi đi kèm một **hành động cụ thể** (Nấu ngay / Cấp đông / Dùng thay thế / Thải bỏ an toàn).

### 1.2. Vòng lặp vận hành cốt lõi (Core Operating Loop)

```text
[ CAPTURE ]       -->  [ UNDERSTAND ]     -->  [ PRIORITISE ]     -->  [ ACT ]         -->  [ RECONCILE ]
Nhập cực nhanh         Giải mã ngữ nghĩa       Hàng đợi chú ý          Gợi ý hành động       Đối soát 1 chạm
(Barcode/OCR/Quick)    (BestBefore vs PAO)    (Now / Soon / Later)    (Cook/Freeze/Use)    (Khớp thực tế)
```

- **Sự thật vật lý duy nhất (Single Source of Truth):** Không gian gia đình thực tế là sự thật duy nhất; hệ thống số chỉ đóng vai trò là "lớp phản chiếu độ trễ thấp" hỗ trợ người dùng ra quyết định.

---

## 2. KIẾN TRÚC HỆ THỐNG (SYSTEM ARCHITECTURE)

Dự án áp dụng kiến trúc **Lõi Vòng Đời Dùng Chung (Shared Lifecycle Core)** kết hợp với **Các Module Chính Sách Từng Miền (Domain Policy Modules)** để đảm bảo khả năng mở rộng không làm vỡ tính toàn vẹn nghiệp vụ:

```text
+-----------------------------------------------------------------------------------+
|                                  USER INTERFACE LAYER                             |
|         Unified "Needs Attention" Queue  |  Rapid Capture Bar  |  1-Tap Triage   |
+-----------------------------------------------------------------------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                             SHARED LIFECYCLE CORE                                 |
|  - Lifecycle Events Bus (Acquired, Opened, Frozen, Consumed, Discarded)           |
|  - State Reconciliation Engine (Physical-to-Digital Sync)                         |
|  - Attention Priority Scorer (Urgency + Location + Household Context)             |
|  - Multi-user Household Sync & Provenance Ledger                                  |
+-----------------------------------------------------------------------------------+
             |                          |                          |
             v                          v                          v
+------------------------+ +------------------------+ +------------------------+
|      FoodPolicy        | |    CosmeticsPolicy     | |     MedicinePolicy     |
|       (Phase 1)        | |       (Phase 2)        | |       (Phase 3)        |
| - Best Before / Use By | | - PAO (Opening Trigger)| | - Safety / Rx Guard    |
| - Opened-on Perish     | | - Formula Sensitivity  | | - Authoritative Return |
| - Fridge/Freezer Shift | | - Risk-Value Tradeoff  | | - Safe Disposal Route  |
+------------------------+ +------------------------+ +------------------------+
```

### Các tầng trách nhiệm (Ownership Boundaries)
- **Lifecycle Core**: Sở hữu định danh vòng đời (`LifecycleItem`), lịch sử sự kiện (`events[]`), trạng thái chú ý (`AttentionState`), và cơ chế đối soát tức thì (`Reconcile`).
- **Domain Policies**: Sở hữu quy tắc tính toán hạn dùng động, cảnh báo ngữ nghĩa đặc thù, và danh mục hành động được phép đề xuất.
- **Presentation (UI Shell)**: Giữ giao diện tinh gọn, tập trung vào hàng đợi *"Cần xử lý hôm nay"*, triệt tiêu việc phải duyệt qua toàn bộ kho dữ liệu.

---

## 3. NGHIÊN CỨU HÀNH VI & PROTO-PERSONAS

Dựa trên tổng hợp nghiên cứu hành vi thực nghiệm đa quốc gia:

| Tiêu chí | Persona 1: Minh (The Pragmatist) | Persona 2: An (The Forgetter) | Persona 3: Lan (The Household Manager) |
| :--- | :--- | :--- | :--- |
| **Đặc trưng** | **Thực dụng & Tiết kiệm chi phí** | **Tiện lợi trên hết & Hay quên** | **Trách nhiệm & Quản trị an toàn** |
| **Động lực chính** | Tiết kiệm tiền bạc, tránh lãng phí thức ăn | Tiện lợi, không muốn tốn công sắp xếp | Bảo vệ an toàn sức khỏe cả gia đình |
| **Hành vi kiểm tra** | Kiểm tra cảm quan, ghét nhập liệu phức tạp | Ít khi chủ động kiểm tra, phản ứng khi đã muộn | Thường xuyên kiểm tra nhãn mác, lo âu |
| **Rào cản lớn nhất** | Bất tiện khi phải duy trì kho số | Cạnh tranh chú ý từ cuộc sống bận rộn | Dữ liệu gia đình bị phân tán, chồng chéo |
| **JTBD (Job To Be Done)** | *"Khi mua đồ về, giúp tôi biết món gì cần ăn trước mà không bắt tôi làm quản kho."* | *"Khi đồ sắp hỏng, nhắc tôi đúng 1 lần với hành động ăn ngay được."* | *"Giúp cả nhà đồng bộ đồ đã mở nắp để không mua thừa hay ăn phải đồ ôi."* |
| **Giá trị cốt lõi** | **Tiết kiệm thời gian & tiền bạc** | **Gia giảm tải nhận thức** | **Kiểm soát & Yên tâm an toàn** |

---

## 4. DANH MỤC YÊU CẦU NGHIỆP VỤ (PRIORITIZED BUSINESS REQUIREMENTS)

| Mã | Mức ưu tiên | Tên yêu cầu | Mô tả chi tiết & Tiêu chuẩn nghiệm thu |
| :--- | :---: | :--- | :--- |
| **BR-A** | **P0** | **Thu nạp cực nhanh (Fast Capture & Provenance)** | Hỗ trợ quét mã vạch (EAN/UPC), nhận diện ngày bằng OCR, hoặc chọn nhanh danh mục thông dụng với hạn mặc định ước lượng. Ghi nhận rõ nguồn gốc dữ liệu (Scan / User manual / Inferred). |
| **BR-B** | **P0** | **Mô hình Vòng đời Đa trạng thái (Lifecycle Model)** | Không chỉ lưu 1 trường ngày chết. Lưu trữ chuỗi sự kiện: Mua (`Acquired`) $\to$ Mở nắp (`Opened`) $\to$ Cấp đông (`Frozen`) $\to$ Tiêu thụ (`Consumed`) $\to$ Thải bỏ (`Discarded`). |
| **BR-C** | **P0** | **Hàng đợi chú ý tập trung (Needs-Attention Queue)** | Giao diện chính phân loại thông minh thành 3 tầng: **Hôm nay (Today)**, **Sắp tới (Soon - 3 ngày)**, và **An toàn (Later)**. Người dùng không phải tìm kiếm qua hàng trăm món. |
| **BR-D** | **P0** | **Đối soát trạng thái 1 chạm (1-Tap Reconciliation)** | Cho phép người dùng xác nhận nhanh tình trạng vật lý bằng 1 nút bấm: *Đã dùng*, *Đã vứt*, *Vẫn còn*, *Đã cấp đông*. Chống tích tụ rác dữ liệu. |
| **BR-E** | **P0** | **Cảnh báo đi kèm hành động (Actionable Alerts)** | Thông báo phải đi liền với hành động khả thi: Gợi ý công thức nấu kết hợp, nút chuyển cấp đông để dời hạn, hoặc đánh dấu đã dùng trực tiếp trên thông báo. |
| **BR-F** | **P0 (Food)** | **Quy tắc đặc thù Thực phẩm (Food Policy)** | Phân biệt rõ rệt giữa *Best Before* (suy giảm phẩm chất) và *Use By* (rủi ro an toàn sinh học). Tự động rút ngắn hạn khi sản phẩm chuyển trạng thái `Opened`. |
| **BR-G** | **P1** | **Chia sẻ & Đồng bộ Hộ gia đình (Household Sync)** | Hỗ trợ nhiều thành viên trong nhà cùng thao tác trên 1 tủ lạnh chung, ghi nhận lịch sử ai đã mở hộp sữa hoặc ai đã dùng hết món gì. |
| **BR-H** | **P1 (Cosmetics)**| **Theo dõi PAO & Mở nắp Mỹ phẩm (Cosmetics PAO)** | Hỗ trợ biểu tượng chiếc hũ mở nắp (6M, 12M, 24M), kích hoạt đồng hồ đếm ngược từ sự kiện bấm mở hộp đầu tiên. |
| **BR-I** | **P2 (Medicine)** | **Quản trị Thuốc & Thải bỏ an toàn (Medicine Safety)** | Phân định thuốc đang điều trị vs thuốc dư thừa; cung cấp hướng dẫn thải bỏ thuốc chuẩn y tế, ngăn ngừa tái sử dụng sai cách. |

---

## 5. THIẾT KẾ MÔ HÌNH DỮ LIỆU (CORE DATA CONTRACTS)

```typescript
// Shared Lifecycle Domain Models (Preview)

export type DomainType = 'food' | 'cosmetics' | 'medicine' | 'warranty';

export type LifecycleEventType = 
  | 'ACQUIRED' 
  | 'OPENED' 
  | 'LOCATION_CHANGED' 
  | 'CONSUMED' 
  | 'DISCARDED';

export interface LifecycleEvent {
  id: string;
  type: LifecycleEventType;
  timestamp: string;
  metadata?: Record<string, unknown>;
  actorId?: string;
}

export type AttentionUrgency = 'CRITICAL_TODAY' | 'APPROACHING_SOON' | 'STABLE_LATER';

export interface LifecycleItem {
  id: string;
  name: string;
  domain: DomainType;
  householdId: string;
  location: 'fridge' | 'freezer' | 'pantry' | 'bathroom' | 'cabinet';
  
  // Nguồn gốc dữ liệu
  provenance: {
    source: 'BARCODE_SCAN' | 'OCR_DATE' | 'USER_MANUAL' | 'CATEGORY_DEFAULT';
    confidenceScore: number;
    initialCaptureDate: string;
  };

  // Lịch sử biến đổi vòng đời
  events: LifecycleEvent[];

  // Trạng thái chú ý được tính toán động
  attentionState: {
    urgency: AttentionUrgency;
    calculatedDeadline: string;
    reasoning: string; // e.g., "Opened 2 days ago (Max safe: 3 days)"
  };

  // Hành động khả thi theo miền
  availableActions: Array<'CONSUME' | 'FREEZE' | 'DISCARD' | 'REPLACE'>;
}

// Food Domain Specific Extension
export interface FoodItemPayload {
  dateType: 'BEST_BEFORE' | 'USE_BY';
  perishabilityTier: 'HIGHLY_PERISHABLE' | 'PERISHABLE' | 'SHELF_STABLE';
  storageGuidelines: {
    ambientShelfDays?: number;
    refrigeratedShelfDays?: number;
    frozenShelfDays?: number;
    afterOpeningRefrigeratedDays?: number;
  };
}
```

---

## 6. LỘ TRÌNH TRIỂN KHAI (ROADMAP)

- [x] **Milestone 0: Nghiên cứu thực nghiệm & Thiết kế kiến trúc (Foundations)**
  - Thu thập bằng chứng thực nghiệm hành vi người tiêu dùng.
  - Xây dựng 3 Proto-Personas và hoàn thiện bộ yêu cầu nghiệp vụ P0.
  - Khởi tạo kiến trúc lõi `Shared Lifecycle Core`.
- [ ] **Milestone 1: Thực phẩm cốt lõi (Food MVP)**
  - Trải nghiệm nhập nhanh: Quét barcode + OCR nhận diện ngày in bao bì.
  - Hàng đợi *"Cần xử lý"* (Today / Soon / Later).
  - Đối soát 1 chạm (1-Tap State Reconciliation).
  - Phân tách ngữ nghĩa *Best Before* vs *Use By*.
- [ ] **Milestone 2: Tương tác Hộ gia đình (Household Sync)**
  - Chia sẻ không gian lưu trữ qua mã gia đình / QR code.
  - Nhật ký hoạt động dùng chung tránh mua trùng lặp.
- [ ] **Milestone 3: Mở rộng Mỹ phẩm (Cosmetics PAO Engine)**
  - Theo dõi hạn mở nắp PAO (6M/12M/24M).
  - Quản lý hạn dùng sản phẩm dưỡng da & trang điểm nhạy cảm.
- [ ] **Milestone 4: Quản lý Thuốc an toàn & Bảo hành (Medicine & Warranty)**
  - Khuyến nghị điểm thu hồi / hướng dẫn thải bỏ thuốc an toàn theo chuẩn y tế.
  - Quản lý hóa đơn và thời hạn bảo hành thiết bị.

---

## 7. QUY CHUẨN KỸ NGHỆ (ENGINEERING EXCELLENCE)

1. **Local-First & Offline Resilience:** Dữ liệu gia đình phải sẵn sàng tức thì ngay cả khi mạng chập chờn tại góc bếp. Đồng bộ nền phi đối xứng.
2. **Minimizing Cognitive Friction:** Mỗi màn hình chỉ giải quyết đúng 1 câu hỏi cốt lõi: *"Món gì đang cần ăn trước?"* hoặc *"Làm sao ghi nhận món này dưới 3 giây?"*.
3. **No Decorative Features:** Tránh các tính năng gamification rườm rà, biểu đồ phức tạp gây tăng gánh nặng sử dụng mà không giải quyết được gốc rễ thất thoát.

---

## 8. TÀI LIỆU THAM KHẢO CHÍNH (BACKBONE EVIDENCE)

- **TotalCtrl / Food App Evaluation (2022)** — *JMIR Formative Research*, e38520. Phân tích nguyên nhân gánh nặng kiểm kê số gây thất bại trong việc duy trì thói quen.
- **LOWINFOOD CozZo Intervention (Horizon Europe)** — Nghiên cứu can thiệp thực tế trên 52 hộ gia đình tại Áo, Phần Lan, Hy Lạp.
- **Wang et al. (2025)** — *Why ignore expiry dates on cosmetics? A qualitative study of perceived risk*, Risk Analysis. Khảo sát sâu về hành vi sử dụng mỹ phẩm quá hạn và hạn mở nắp.
- **Kelly et al. (2018)** — *‘You don’t throw these things around’: Understanding medicine storage and disposal in homes*. Nghiên cứu tích trữ thuốc và hạn dùng trong gia đình.
- **US FDA & USDA Guidelines** — Báo cáo hướng dẫn phân biệt nhãn ngày thực phẩm (*Product Dating*) và các yếu tố gây lãng phí thực phẩm.
