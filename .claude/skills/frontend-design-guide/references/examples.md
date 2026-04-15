# Frontend Design Guide — Bad/Good 코드 예시

SKILL.md의 규칙 번호에 대응하는 코드 예시 모음. 설계 상담 시 해당 규칙의 예시를 참조한다.

---

## 1. 가독성

### 1-1. 같이 실행되지 않는 코드 분리

```tsx
// Bad — 뷰어/에디터 로직이 하나에 혼재
function SubmitButton() {
  const isViewer = useRole() === 'viewer';
  useEffect(() => {
    if (isViewer) return;
    showButtonAnimation();
  }, [isViewer]);
  return isViewer
    ? <TextButton disabled>Submit</TextButton>
    : <Button type="submit">Submit</Button>;
}

// Good — 분기가 하나로 통합, 각 컴포넌트는 단일 경로만
function SubmitButton() {
  const isViewer = useRole() === 'viewer';
  return isViewer ? <ViewerSubmitButton /> : <AdminSubmitButton />;
}

function ViewerSubmitButton() {
  return <TextButton disabled>Submit</TextButton>;
}

function AdminSubmitButton() {
  useEffect(() => { showButtonAnimation(); }, []);
  return <Button type="submit">Submit</Button>;
}
```

### 1-2. 구현 상세 추상화

```tsx
// Bad — 로그인 확인 + 리다이렉트 로직이 컴포넌트 내부에 노출
function LoginStartPage() {
  useCheckLogin({
    onChecked: (status) => {
      if (status === 'LOGGED_IN') { location.href = '/home'; }
    },
  });
  return <>{/* 로그인 관련 컴포넌트 */}</>;
}

// Good — AuthGuard가 인증 로직을 캡슐화
function App() {
  return <AuthGuard><LoginStartPage /></AuthGuard>;
}

function AuthGuard({ children }: { children: React.ReactNode }) {
  const status = useCheckLoginStatus();
  useEffect(() => {
    if (status === 'LOGGED_IN') { location.href = '/home'; }
  }, [status]);
  return status !== 'LOGGED_IN' ? <>{children}</> : null;
}
```

### 1-3. 로직 종류별 함수 쪼개기

```typescript
// Bad — 5개 쿼리 파라미터를 하나의 Hook에서 관리
export function usePageState() {
  const [query, setQuery] = useQueryParams({
    cardId: NumberParam, statementId: NumberParam,
    dateFrom: DateParam, dateTo: DateParam, statusList: ArrayParam,
  });
  return useMemo(() => ({
    values: { /* 5개 파라미터 변환 */ },
    controls: { /* 5개 setter */ },
  }), [query, setQuery]);
}

// Good — 개별 Hook으로 분리
export function useCardIdQueryParam() {
  const [cardId, _setCardId] = useQueryParam('cardId', NumberParam);
  const setCardId = useCallback(
    (newCardId: number) => _setCardId({ cardId: newCardId }, 'replaceIn'),
    [_setCardId],
  );
  return [cardId ?? undefined, setCardId] as const;
}
```

### 1-4. 복잡한 조건에 이름 붙이기

```typescript
const matchedProducts = products.filter((product) => {
  const isSameCategory = product.categories.some(
    (category) => category.id === targetCategory.id,
  );
  const isPriceInRange = product.prices.some(
    (price) => price >= minPrice && price <= maxPrice,
  );
  return isSameCategory && isPriceInRange;
});
```

### 1-5. 매직 넘버에 이름 붙이기

```typescript
// Bad
async function onLikeClick() {
  await postLike(url);
  await delay(300);
  await refetchPostLike();
}

// Good
const ANIMATION_DELAY_MS = 300;
async function onLikeClick() {
  await postLike(url);
  await delay(ANIMATION_DELAY_MS);
  await refetchPostLike();
}
```

### 1-6. 시점 이동 줄이기

```tsx
// Bad — policy.canInvite → getPolicyByRole() → POLICY_SET 3번 시점 이동
function Page() {
  const user = useUser();
  const policy = getPolicyByRole(user.role);
  return <Button disabled={!policy.canInvite}>Invite</Button>;
}

// Good — 인라인 객체로 시점 이동 0회
function Page() {
  const user = useUser();
  const policy = {
    admin: { canInvite: true, canView: true },
    viewer: { canInvite: false, canView: true },
  }[user.role];

  return (
    <div>
      <Button disabled={!policy.canInvite}>Invite</Button>
      <Button disabled={!policy.canView}>View</Button>
    </div>
  );
}
```

### 1-7. 삼항 연산자 단순하게

```typescript
// Bad
const status = A조건 && B조건 ? 'BOTH' : A조건 || B조건 ? (A조건 ? 'A' : 'B') : 'NONE';

// Good
const status = (() => {
  if (A조건 && B조건) return 'BOTH';
  if (A조건) return 'A';
  if (B조건) return 'B';
  return 'NONE';
})();
```

### 1-8. Early Return

```tsx
// Bad
function ProductCard({ product }: Props) {
  if (product) {
    if (product.isAvailable) {
      if (product.price > 0) { return <Card>{product.name}</Card>; }
      else { return <FreeCard />; }
    } else { return <SoldOut />; }
  } else { return null; }
}

// Good
function ProductCard({ product }: Props) {
  if (!product) return null;
  if (!product.isAvailable) return <SoldOut />;
  if (product.price === 0) return <FreeCard />;
  return <Card>{product.name}</Card>;
}
```

### 1-9. 컴포넌트 내부 선언 순서 통일

```tsx
function ProfilePage() {
  // 1. hooks
  const [name, setName] = useState('');
  const { data: user } = useUser();

  // 2. derived state
  const isValid = name.length > 0 && name.length <= 20;

  // 3. handlers
  const handleSubmit = () => { updateProfile(name); };

  // 4. effects
  useEffect(() => { if (user) setName(user.name); }, [user]);

  // 5. early returns
  if (!user) return <Skeleton />;

  // 6. render
  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <button disabled={!isValid}>저장</button>
    </form>
  );
}
```

### 1-10. JSX 인라인 함수 추출

```tsx
// Bad
<li onClick={() => {
  trackEvent('item_click', { id: item.id });
  router.push(`/items/${item.id}`);
  setLastViewed(item.id);
}}>

// Good
const handleItemClick = (itemId: string) => {
  trackEvent('item_click', { id: itemId });
  router.push(`/items/${itemId}`);
  setLastViewed(itemId);
};
<li onClick={() => handleItemClick(item.id)}>
```

### 1-11. Boolean Props 긍정형

```tsx
// Bad — 조건문에서 이중 부정 발생
<Modal isHidden={!shouldShow} />
// 사용부: {!isHidden && <Content />} → 이중 부정

// Good — 긍정형으로 단일 판단
<Modal visible={shouldShow} />
// 사용부: {visible && <Content />}

// Bad — 의미 반전 prop
<Input isReadOnly={!canEdit} />

// Good — 의미 직관적
<Input editable={canEdit} />
```

---

## 2. 예측 가능성

### 2-1. 고유하고 설명적인 이름

```typescript
// Bad — 라이브러리의 http와 래퍼의 http가 동일 이름
export const http = {
  async get(url: string) {
    const token = await fetchToken();
    return httpLibrary.get(url, { headers: { Authorization: `Bearer ${token}` } });
  },
};

// Good — 이름으로 역할 차이를 명확히 구분
export const httpService = {
  async getWithAuth(url: string) {
    const token = await fetchToken();
    return httpLibrary.get(url, { headers: { Authorization: `Bearer ${token}` } });
  },
};

// Bad — logEvent가 콘솔 로그인지 서버 전송인지 모호
function logEvent(event: string) { return sendToServer(event); }

// Good — 역할이 이름에 드러남
function sendAnalyticsEvent(eventName: string) { return analyticsClient.send(eventName); }
```

### 2-2. 반환 타입 통일

```typescript
// Bad
function useUser() { return useQuery({ queryKey: ['user'], queryFn: fetchUser }); }
function useServerTime() {
  const query = useQuery({ queryKey: ['time'], queryFn: fetchTime });
  return query.data;  // 일관성 없음
}

// Good — 둘 다 query 객체 반환
function useUser() { return useQuery({ queryKey: ['user'], queryFn: fetchUser }); }
function useServerTime() { return useQuery({ queryKey: ['time'], queryFn: fetchTime }); }

// 검증 함수도 일관된 타입
type ValidationResult = { ok: true } | { ok: false; reason: string };
```

### 2-3. 숨은 로직 드러내기

```typescript
// Bad — fetchBalance 안에 로깅이 숨어있음
async function fetchBalance(): Promise<number> {
  const balance = await http.get<number>('...');
  logging.log('balance_fetched');  // 숨은 부작용!
  return balance;
}

// Good — 로깅은 호출부에서 명시적으로
const handleRefresh = async () => {
  const balance = await fetchBalance();
  logging.log('balance_fetched');
  await syncBalance(balance);
};
```

### 2-4. 파라미터 객체 전환

```typescript
// Bad
sendNotification('user-1', 'email', true, 3, 'high');

// Good
sendNotification({
  userId: 'user-1',
  channel: 'email',
  isUrgent: true,
  retryCount: 3,
  priority: 'high',
});
```

### 2-5. Discriminated Union

```tsx
// Bad
type Props = { status: 'loading' | 'success' | 'error'; data?: ResponseData; error?: Error; };

// Good
type Props =
  | { status: 'loading' }
  | { status: 'success'; data: ResponseData }
  | { status: 'error'; error: Error };

function AsyncResult(props: Props) {
  switch (props.status) {
    case 'loading': return <Spinner />;
    case 'success': return <DataView data={props.data} />;
    case 'error':   return <ErrorView error={props.error} />;
  }
}
```

### 2-6. useEffect 의도 명시

```tsx
// Bad
useEffect(() => {
  if (isReady && itemId && !hasProcessed) { submitForm(); setHasProcessed(true); }
}, [isReady, itemId, hasProcessed]);

// Good
function useOnceWhen(condition: boolean, callback: () => void) {
  const hasRun = useRef(false);
  useEffect(() => {
    if (condition && !hasRun.current) { hasRun.current = true; callback(); }
  }, [condition, callback]);
}

useOnceWhen(isReady && !!itemId, () => submitForm());
```

### 2-7. 핸들러 네이밍 일관성

```tsx
type Props = { onFormSubmit: () => void; };

function ContactForm({ onFormSubmit }: Props) {
  const handleSubmit = () => { validate(); onFormSubmit(); };
  return <Button onClick={handleSubmit} />;
}
```

---

## 3. 응집성

### 3-1. Co-location

```
# Bad — 모듈 종류별 분류
src/components/ hooks/ utils/ constants/ contexts/

# Good — 도메인 기반 구조
src/
  components/ hooks/     (공통)
  domains/
    Auth/     components/ hooks/ utils/
    Dashboard/ components/ hooks/ utils/
```

### 3-2. 폼 응집도 (필드 수준)

```tsx
const NameField = ({ register, error }: FieldProps) => (
  <div>
    <input {...register('name', { required: '이름을 입력해주세요.' })} />
    {error && <p>{error.message}</p>}
  </div>
);
```

### 3-3. Feature Hook

```tsx
// Bad — 300줄 컴포넌트에 로직 흩어짐
function OrderPage() {
  const [step, setStep] = useState<Step>('form');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const handleOrder = async () => { /* 20줄 */ };
}

// Good — 도메인 로직 응집
function useOrder() {
  const [step, setStep] = useState<Step>('form');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [error, setError] = useState<Error | null>(null);
  const submit = async () => { /* 로직 */ };
  return { step, setStep, isSubmitting, error, submit };
}

function OrderPage() {
  const order = useOrder();
  return <OrderForm {...order} />;
}
```

### 3-4. 컴포넌트 크기 제한

```tsx
// Bad — 모놀리식 (상품+배송+가격+액션 전부)
function CheckoutPage() {
  // 상품 정보 로직 50줄
  // 배송 정보 로직 50줄
  // 가격 계산 로직 30줄
  // JSX 120줄 = 총 250줄+
}

// Good — 역할별 분리 (각 컴포넌트가 한 화면에 들어옴)
function CheckoutPage() {
  return (
    <>
      <OrderSummary orderId={id} />
      <ShippingForm onComplete={setAddress} />
      <PriceSummary price={price} discount={discount} />
      <SubmitBar onSubmit={handleSubmit} />
    </>
  );
}
```

### 3-5. API 변환 응집

```tsx
// Bad — 컴포넌트에서 API 응답 직접 변환
function UserList() {
  const { data } = useQuery(['users'], fetchUsers);
  const users = data?.items.map((item) => ({
    id: item.user_id, name: item.user_nm, age: Number(item.age_val),
  }));
}

// Good — API 레이어에서 변환 완료
export async function fetchUsers(): Promise<User[]> {
  const { data } = await instance.get<RawUserResponse>('/users');
  return data.items.map(toUser);
}
```

---

## 4. 결합도

### 4-1. 중복 코드 허용

```typescript
// Bad — 섣부른 공통화
export const useOpenMaintenanceBottomSheet = () => {
  const maintenanceBottomSheet = useMaintenanceBottomSheet();
  const logger = useLogger();
  return async (info: MaintenanceInfo) => {
    logger.log('점검 바텀시트 열림');
    const result = await maintenanceBottomSheet.open(info);
    if (result) { logger.log('알림받기 클릭'); }
    closeView();
  };
};
// 문제: 페이지별 로깅 다를 때 파급, closeView 불필요한 페이지 존재, UI 차이 대응 불가
```

### 4-2. Props Drilling 제거

**방법 A — 조합 패턴 (children):**
```tsx
// Good — children으로 중간 단계 전달 제거
function ItemEditModal({ open, onClose, children }) {
  const [keyword, setKeyword] = useState('');
  return (
    <Modal open={open} onClose={onClose}>
      <ItemEditBody keyword={keyword} onKeywordChange={setKeyword}>
        {children}
      </ItemEditBody>
    </Modal>
  );
}
```

**방법 B — Context API:**
```tsx
const ItemEditContext = createContext<ItemEditState | null>(null);

function ItemEditProvider({ children, items, recommendedItems }) {
  const value = useMemo(() => ({ items, recommendedItems }), [items, recommendedItems]);
  return <ItemEditContext.Provider value={value}>{children}</ItemEditContext.Provider>;
}
```

**모달 합성 예시:**
```tsx
// Bad — props 10개
<SettingsModal isOpen={isOpen} theme={theme} language={lang}
  onConfirm={handleConfirm} onCancel={handleCancel}
  showAdvanced={true} advancedOptions={options} />

// Good — children 합성으로 Modal은 껍데기만
<Modal isOpen={isOpen} onClose={handleCancel}>
  <ThemeSelector theme={theme} onChange={setTheme} />
  <LanguagePicker lang={lang} onChange={setLang} />
  <Modal.Footer>
    <Button onClick={handleCancel}>취소</Button>
    <Button onClick={handleConfirm}>확인</Button>
  </Modal.Footer>
</Modal>
```

### 4-3. 외부 라이브러리 래핑

```tsx
// Bad — 30곳에서 dayjs 직접 import
import dayjs from 'dayjs';
const formatted = dayjs(date).format('YYYY.MM.DD');

// Good — 래퍼로 교체 비용 1곳
import dayjs from 'dayjs';
export function formatDate(date: string | Date, pattern = 'YYYY.MM.DD') {
  return dayjs(date).format(pattern);
}
```

### 4-4. 형제 간 공유 상태 추출

```tsx
// Bad — 부모가 형제 간 통신 중계
function ParentPage() {
  const [selectedItem, setSelectedItem] = useState(null);
  return (<>
    <ItemGrid onSelect={setSelectedItem} />
    <DetailPanel item={selectedItem} />
  </>);
}

// Good — 전용 store로 직접 구독
const useItemSelection = create<ItemSelectionStore>((set) => ({
  selectedItem: null,
  selectItem: (item: Item) => set({ selectedItem: item }),
}));
```

### 4-5. API 의존성 역전

```tsx
// Bad — 훅이 URL/응답 구조에 직접 결합
function usePostDetail(id: string) {
  return useQuery(['post', id], () =>
    axios.get(`/api/v2/posts/${id}`).then((res) => res.data.result));
}

// Good — API 함수를 통해 간접 접근
export const postApi = {
  getDetail: (id: string) => instance.get<PostDetail>(`/posts/${id}`).then((r) => r.data),
};
function usePostDetail(id: string) {
  return useQuery(['post', id], () => postApi.getDetail(id));
}
```

### 4-6. Headless 패턴

```tsx
// 로직 훅
function useDropdown<T>(items: T[]) {
  const [isOpen, setIsOpen] = useState(false);
  const [selected, setSelected] = useState<T | null>(null);
  const ref = useRef<HTMLDivElement>(null);
  useEffect(() => { /* 외부 클릭 닫기 */ }, []);
  return { isOpen, selected, ref, toggle: () => setIsOpen((v) => !v), select: setSelected };
}

// 렌더링 컴포넌트 — 순수 UI만
function CategoryDropdown({ items }: Props) {
  const dropdown = useDropdown(items);
  return (
    <div ref={dropdown.ref}>
      <button onClick={dropdown.toggle}>{dropdown.selected ?? '선택'}</button>
      {dropdown.isOpen && items.map((item) => (
        <div key={item.id} onClick={() => dropdown.select(item)}>{item.name}</div>
      ))}
    </div>
  );
}
```
