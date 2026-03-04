---
name: tdd
description: 测试驱动开发专家。强制执行先写测试的方法论，引导红-绿-重构循环，涵盖单元测试、集成测试和 E2E 测试，确保 80% 以上的测试覆盖率。
---

# 测试驱动开发（TDD）

强制执行测试驱动开发工作流：先搭建接口，首先生成测试，然后实施最小代码通过，确保 80% 以上的覆盖率。

## TDD 循环

```
红 → 绿 → 重构 → 重复

红：编写失败的测试
绿：编写最小代码通过
重构：改进代码，保持测试通过
重复：下一个功能/场景
```

## 工作流程

### 1. 先写测试（红）
写一个描述预期行为的失败测试。

### 2. 运行测试 -- 验证它失败
```bash
npm test
```

### 3. 写最小实现（绿）
只需要足够的代码让测试通过。

### 4. 运行测试 -- 验证它通过

### 5. 重构（改进）
移除重复，改进名称，优化 -- 测试必须保持绿色。

### 6. 验证覆盖率
```bash
npm run test:coverage
# 要求：80% 以上的分支、函数、行、语句
```

## 测试类型

| 类型 | 测试什么 | 何时 |
|------|-------------|------|
| **单元** | 隔离的单个函数 | 始终 |
| **集成** | API 端点、数据库操作 | 始终 |
| **E2E** | 关键用户流程（Playwright） | 关键路径 |

## 必须测试的边缘情况

1. **Null/Undefined** 输入
2. **空** 数组/字符串
3. **无效类型** 传递
4. **边界值**（最小/最大）
5. **错误路径**（网络失败、数据库错误）
6. **竞态条件**（并发操作）
7. **大数据**（10k+ 项目的性能）
8. **特殊字符**（Unicode、emoji、SQL 字符）

## 单元测试示例

### 步骤 1：定义接口

```typescript
// lib/liquidity.ts
export interface MarketData {
  totalVolume: number
  bidAskSpread: number
  activeTraders: number
  lastTradeTime: Date
}

export function calculateLiquidityScore(market: MarketData): number {
  throw new Error('Not implemented')
}
```

### 步骤 2：编写失败的测试

```typescript
// lib/liquidity.test.ts
describe('calculateLiquidityScore', () => {
  it('should return high score for liquid market', () => {
    const market = {
      totalVolume: 100000,
      bidAskSpread: 0.01,
      activeTraders: 500,
      lastTradeTime: new Date()
    }
    const score = calculateLiquidityScore(market)
    expect(score).toBeGreaterThan(80)
  })

  it('should handle edge case: zero volume', () => {
    const market = { totalVolume: 0, bidAskSpread: 0, activeTraders: 0, lastTradeTime: new Date() }
    expect(calculateLiquidityScore(market)).toBe(0)
  })
})
```

### 步骤 3：实施最小代码

```typescript
export function calculateLiquidityScore(market: MarketData): number {
  if (market.totalVolume === 0) return 0
  const volumeScore = Math.min(market.totalVolume / 1000, 100)
  const spreadScore = Math.max(100 - (market.bidAskSpread * 1000), 0)
  const traderScore = Math.min(market.activeTraders / 10, 100)
  return volumeScore * 0.4 + spreadScore * 0.3 + traderScore * 0.3
}
```

---

## 后端测试示例 (Java)

### 单元测试 (JUnit 5)

```java
// src/main/java/com/app/service/OrderService.java
@Service
public class OrderService {

    public BigDecimal calculateTotal(List<OrderItem> items) {
        throw new UnsupportedOperationException("Not implemented");
    }
}

// src/test/java/com/app/service/OrderServiceTest.java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @InjectMocks
    private OrderService orderService;

    @Test
    void shouldCalculateTotalCorrectly() {
        List<OrderItem> items = List.of(
            new OrderItem("P1", 2, new BigDecimal("10.00")),
            new OrderItem("P2", 1, new BigDecimal("5.00"))
        );

        BigDecimal total = orderService.calculateTotal(items);

        assertEquals(new BigDecimal("25.00"), total);
    }

    @Test
    void shouldReturnZeroForEmptyList() {
        BigDecimal total = orderService.calculateTotal(List.of());
        assertEquals(BigDecimal.ZERO, total);
    }

    @Test
    void shouldThrowExceptionForNullItems() {
        assertThrows(NullPointerException.class,
            () -> orderService.calculateTotal(null));
    }
}
```

### Spring Boot 集成测试

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Test
    void shouldCreateUser() throws Exception {
        when(userService.createUser(any())).thenReturn(1L);

        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"email\":\"test@example.com\",\"name\":\"Test\"}"))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1));
    }

    @Test
    void shouldReturn404WhenUserNotFound() throws Exception {
        when(userService.getUser(999L)).thenReturn(Optional.empty());

        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound());
    }
}
```

### 命令

```bash
# Maven
./mvnw test                    # 运行测试
./mvnw test -Dtest=OrderServiceTest  # 运行特定测试
./mvnw jacoco:report           # 生成覆盖率报告

# Gradle
./gradlew test                 # 运行测试
./gradlew test --tests "*.OrderServiceTest"  # 特定测试
./gradlew jacocoTestReport     # 覆盖率报告
```

---

# E2E 测试（Playwright）

关键用户流程的端到端测试。

## 命令参考

```bash
npx playwright test                        # 运行所有 E2E 测试
npx playwright test tests/auth.spec.ts     # 运行特定文件
npx playwright test --headed               # 查看浏览器
npx playwright test --debug                # 使用检查器调试
npx playwright test --trace on             # 带追踪运行
npx playwright show-report                 # 查看 HTML 报告
```

## 测试文件组织

```
tests/
├── e2e/
│   ├── auth/
│   │   ├── login.spec.ts
│   │   └── register.spec.ts
│   └── features/
│       ├── search.spec.ts
│       └── create.spec.ts
├── fixtures/
│   └── auth.ts
└── playwright.config.ts
```

## 页面对象模型（POM）

```typescript
import { Page, Locator } from '@playwright/test'

export class ItemsPage {
  readonly page: Page
  readonly searchInput: Locator
  readonly itemCards: Locator

  constructor(page: Page) {
    this.page = page
    this.searchInput = page.locator('[data-testid="search-input"]')
    this.itemCards = page.locator('[data-testid="item-card"]')
  }

  async goto() {
    await this.page.goto('/items')
    await this.page.waitForLoadState('networkidle')
  }

  async search(query: string) {
    await this.searchInput.fill(query)
    await this.page.waitForResponse(resp => resp.url().includes('/api/search'))
  }
}
```

## E2E 测试结构

```typescript
import { test, expect } from '@playwright/test'
import { ItemsPage } from '../../pages/ItemsPage'

test.describe('Item Search', () => {
  let itemsPage: ItemsPage

  test.beforeEach(async ({ page }) => {
    itemsPage = new ItemsPage(page)
    await itemsPage.goto()
  })

  test('should search by keyword', async ({ page }) => {
    await itemsPage.search('test')
    await expect(itemsPage.itemCards.first()).toContainText(/test/i)
  })

  test('should handle no results', async ({ page }) => {
    await itemsPage.search('xyznonexistent123')
    await expect(page.locator('[data-testid="no-results"]')).toBeVisible()
  })
})
```

## Playwright 配置

```typescript
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  reporter: [['html'], ['junit', { outputFile: 'results.xml' }]],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
  ],
})
```

## 不稳定测试处理

```typescript
// 隔离不稳定测试
test('flaky: complex search', async ({ page }) => {
  test.fixme(true, 'Flaky - Issue #123')
})

// 识别不稳定性
// npx playwright test --repeat-each=10
```

**常见原因和修复：**

```typescript
// 竞态条件 - 使用自动等待定位器
await page.locator('[data-testid="button"]').click()  // 好
await page.click('[data-testid="button"]')            // 坏

// 网络时序 - 等待特定条件
await page.waitForResponse(resp => resp.url().includes('/api/data'))  // 好
await page.waitForTimeout(5000)                                        // 坏
```

## 质量检查清单

- [ ] 所有公共函数都有单元测试
- [ ] 所有 API 端点都有集成测试
- [ ] 关键用户流程有 E2E 测试
- [ ] 覆盖边缘情况（null、空、无效）
- [ ] 测试错误路径（不只是快乐路径）
- [ ] 对外部依赖使用 mock
- [ ] 测试是独立的（无共享状态）
- [ ] 覆盖率 80%+

## 覆盖率要求

- 所有代码 **最低 80%**
- 以下需要 **100%**：财务计算、认证逻辑、安全关键代码

## 反模式

- 测试实现细节而不是行为
- 测试相互依赖（共享状态）
- 断言太少
- 不 mock 外部依赖
- 使用 `waitForTimeout` 而不是等待条件
