import { useState, type FormEvent } from 'react';
import { Navigate, NavLink, Route, Routes, useNavigate } from 'react-router-dom';
import {
  ArrowDownLeft,
  ArrowUpRight,
  Banknote,
  Bell,
  ChevronRight,
  Copy,
  CreditCard,
  Eye,
  EyeOff,
  FileImage,
  HelpCircle,
  Home,
  Landmark,
  LogOut,
  Percent,
  Plus,
  ReceiptText,
  ShieldCheck,
  Smartphone,
  TrendingUp,
  UserCircle2,
  Wallet,
  Zap,
} from 'lucide-react';
import { toast, Toaster } from 'sonner';

type Role = 'admin' | 'user';

type UserProfile = {
  id: string;
  name: string;
  phone: string;
  inviteCode: string;
  password: string;
  role: Role;
};

type DepositItem = {
  id: string;
  amount: number;
  reward: number;
  status: 'Pending' | 'Approved' | 'Rejected';
  utr: string;
  date: string;
};

type WithdrawalItem = {
  id: string;
  amount: number;
  mode: 'UPI ID' | 'Bank Account';
  details: string;
  status: 'Pending' | 'Approved' | 'Rejected';
  date: string;
};

type TransactionActivity = {
  id: string;
  title: string;
  subtitle: string;
  amount: string;
  type: 'income' | 'expense' | 'reward';
  date: string;
};

type PaymentOrder = {
  id: string;
  amount: number;
  reward: number;
  label: string;
};

const generateMintId = () => {
  const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
  let id = 'MPFD';
  for (let i = 0; i < 8; i += 1) {
    id += chars[Math.floor(Math.random() * chars.length)];
  }
  return id;
};

const formatCurrency = (value: number) =>
  new Intl.NumberFormat('en-IN', {
    style: 'currency',
    currency: 'INR',
    maximumFractionDigits: 0,
  }).format(value);

const defaultUser: UserProfile = {
  id: 'MPFD01DC53',
  name: 'Aman Verma',
  phone: '+91 98765 43210',
  inviteCode: 'WELCOME10',
  password: 'demo123',
  role: 'admin',
};

const defaultDeposits: DepositItem[] = [
  { id: 'D-201', amount: 25000, reward: 1050, status: 'Approved', utr: 'UTR987654321001', date: 'Today, 9:20 AM' },
  { id: 'D-210', amount: 4200, reward: 176, status: 'Pending', utr: 'UTR987654321045', date: 'Today, 11:10 AM' },
  { id: 'D-212', amount: 12000, reward: 504, status: 'Approved', utr: 'UTR987654321099', date: 'Yesterday' },
  { id: 'D-220', amount: 7800, reward: 328, status: 'Rejected', utr: 'UTR987654321120', date: '2 days ago' },
];

const defaultWithdrawals: WithdrawalItem[] = [
  { id: 'W-178', amount: 4000, mode: 'UPI ID', details: 'aman@upi', status: 'Pending', date: 'Today, 10:30 AM' },
  { id: 'W-169', amount: 2200, mode: 'Bank Account', details: 'HDFC • 0218314324', status: 'Approved', date: 'Yesterday' },
  { id: 'W-160', amount: 850, mode: 'UPI ID', details: 'aman.pay@oksbi', status: 'Rejected', date: '2 days ago' },
];

const defaultActivities: TransactionActivity[] = [
  { id: 'A1', title: 'Deposit approved', subtitle: 'UPI transfer • UTR 987654321001', amount: '+₹25,000', type: 'income', date: 'Today' },
  { id: 'A2', title: 'Withdrawal to bank', subtitle: 'HDFC • 0218314324', amount: '-₹2,200', type: 'expense', date: 'Yesterday' },
  { id: 'A3', title: 'Reward bonus', subtitle: 'Cashback 4.2%', amount: '+₹250', type: 'reward', date: '2 days ago' },
  { id: 'A4', title: 'Deposit paid', subtitle: 'UPI transfer • UTR 987654321099', amount: '+₹12,000', type: 'income', date: '3 days ago' },
];

const paymentOrders: PaymentOrder[] = [
  { id: 'P1', amount: 250, reward: 10.5, label: '₹100 - ₹500' },
  { id: 'P2', amount: 650, reward: 27.3, label: '₹500 - ₹2,000' },
  { id: 'P3', amount: 1500, reward: 63, label: '₹500 - ₹2,000' },
  { id: 'P4', amount: 3200, reward: 134.4, label: '₹2,000 - ₹10,000' },
  { id: 'P5', amount: 6400, reward: 268.8, label: '₹2,000 - ₹10,000' },
  { id: 'P6', amount: 9800, reward: 411.6, label: '₹2,000 - ₹10,000' },
];

const navItems = [
  { label: 'Home', to: '/', icon: Home },
  { label: 'Payment', to: '/payment', icon: CreditCard },
  { label: 'Withdraw', to: '/withdraw', icon: Wallet },
  { label: 'Statistics', to: '/statistics', icon: TrendingUp },
  { label: 'My', to: '/my', icon: UserCircle2 },
];

function App() {
  const [user, setUser] = useState<UserProfile>(defaultUser);
  const [isAuthenticated, setIsAuthenticated] = useState(true);
  const [showBalance, setShowBalance] = useState(true);
  const [deposits, setDeposits] = useState<DepositItem[]>(defaultDeposits);
  const [withdrawals, setWithdrawals] = useState<WithdrawalItem[]>(defaultWithdrawals);
  const [activities, setActivities] = useState<TransactionActivity[]>(defaultActivities);

  const spendableBalance =
    deposits
      .filter((deposit) => deposit.status === 'Approved')
      .reduce((sum, deposit) => sum + deposit.amount + deposit.reward, 0) +
    12000 -
    withdrawals
      .filter((withdrawal) => withdrawal.status === 'Pending' || withdrawal.status === 'Approved')
      .reduce((sum, withdrawal) => sum + withdrawal.amount, 0);

  const approvedDepositsTotal = deposits
    .filter((deposit) => deposit.status === 'Approved')
    .reduce((sum, deposit) => sum + deposit.amount, 0);

  const totalRewards = deposits
    .filter((deposit) => deposit.status === 'Approved')
    .reduce((sum, deposit) => sum + deposit.reward, 0);

  const pendingDeposits = deposits.filter((deposit) => deposit.status === 'Pending').length;
  const pendingWithdrawals = withdrawals.filter((withdrawal) => withdrawal.status === 'Pending').length;

  const onLogin = (name: string, phone: string, password: string, isSignup: boolean) => {
    const normalizedPhone = phone.startsWith('+91') ? phone : `+91 ${phone}`;
    const finalUser: UserProfile = {
      id: generateMintId(),
      name,
      phone: normalizedPhone,
      inviteCode: 'NEWUSER',
      password,
      role: isSignup ? 'user' : 'admin',
    };

    setUser(finalUser);
    setIsAuthenticated(true);
    toast.success(`Welcome back, ${name}!`);
  };

  const handlePaymentSubmit = (amount: number, utr: string) => {
    const reward = Number((amount * 0.042).toFixed(2));
    const nextEntry: DepositItem = {
      id: `D-${Math.floor(Math.random() * 10000)}`,
      amount,
      reward,
      status: 'Pending',
      utr,
      date: 'Just now',
    };

    setDeposits((prev) => [nextEntry, ...prev]);
    setActivities((prev) => [
      {
        id: `A-${Date.now()}`,
        title: 'Deposit pending review',
        subtitle: `UTR ${utr}`,
        amount: `+₹${amount.toLocaleString('en-IN')}`,
        type: 'income',
        date: 'Just now',
      },
      ...prev,
    ]);
    toast.success('Deposit request placed successfully');
  };

  const handleWithdrawalSubmit = (amount: number, mode: 'UPI ID' | 'Bank Account', details: string) => {
    if (amount < 100) {
      toast.error('Minimum withdrawal amount is ₹100');
      return;
    }
    if (amount > spendableBalance) {
      toast.error('Withdrawal amount exceeds available balance');
      return;
    }

    setWithdrawals((prev) => [
      {
        id: `W-${Math.floor(Math.random() * 10000)}`,
        amount,
        mode,
        details,
        status: 'Pending',
        date: 'Just now',
      },
      ...prev,
    ]);
    toast.success('Withdrawal request reserved successfully');
  };

  const handleLogout = () => {
    setIsAuthenticated(false);
    toast.info('You have been logged out');
  };

  return (
    <div className="mobile-shell">
      <Toaster position="top-right" richColors theme="light" />
      <Routes>
        {!isAuthenticated ? (
          <>
            <Route path="*" element={<AuthPage onLogin={onLogin} />} />
          </>
        ) : (
          <>
            <Route
              path="/"
              element={
                <AppShell
                  user={user}
                  showBalance={showBalance}
                  setShowBalance={setShowBalance}
                  spendableBalance={spendableBalance}
                  approvedDepositsTotal={approvedDepositsTotal}
                  totalRewards={totalRewards}
                  pendingDeposits={pendingDeposits}
                  pendingWithdrawals={pendingWithdrawals}
                  activities={activities}
                  onLogout={handleLogout}
                />
              }
            >
              <Route index element={<HomePage showBalance={showBalance} setShowBalance={setShowBalance} spendableBalance={spendableBalance} approvedDepositsTotal={approvedDepositsTotal} totalRewards={totalRewards} activities={activities} />} />
              <Route path="payment" element={<PaymentPage onSubmit={handlePaymentSubmit} />} />
              <Route path="withdraw" element={<WithdrawPage spendableBalance={spendableBalance} onSubmit={handleWithdrawalSubmit} withdrawals={withdrawals} />} />
              <Route path="statistics" element={<StatisticsPage spendableBalance={spendableBalance} approvedDepositsTotal={approvedDepositsTotal} totalRewards={totalRewards} withdrawals={withdrawals} deposits={deposits} />} />
              <Route path="my" element={<MyPage user={user} onLogout={handleLogout} />} />
              <Route path="admin" element={user.role === 'admin' ? <AdminPage deposits={deposits} withdrawals={withdrawals} /> : <Navigate to="/" replace />} />
              <Route path="*" element={<Navigate to="/" replace />} />
            </Route>
          </>
        )}
      </Routes>
    </div>
  );
}

function AppShell({
  user,
  showBalance,
  setShowBalance,
  spendableBalance,
  approvedDepositsTotal,
  totalRewards,
  pendingDeposits,
  pendingWithdrawals,
  activities,
  onLogout,
}: {
  user: UserProfile;
  showBalance: boolean;
  setShowBalance: (value: boolean) => void;
  spendableBalance: number;
  approvedDepositsTotal: number;
  totalRewards: number;
  pendingDeposits: number;
  pendingWithdrawals: number;
  activities: TransactionActivity[];
  onLogout: () => void;
}) {
  const location = window.location.pathname;

  return (
    <>
      <div className="min-h-screen bg-slate-100 pb-24">
        <header className="sticky top-0 z-20 border-b border-slate-200 bg-slate-50/80 px-4 py-4 backdrop-blur">
          <div className="mx-auto flex max-w-md items-center justify-between">
            <div className="flex items-center gap-2">
              <div className="flex h-10 w-10 items-center justify-center rounded-2xl bg-green-500 text-white shadow-lg shadow-green-200">
                <LeafIcon />
              </div>
              <div>
                <p className="text-xs text-slate-500">Welcome</p>
                <h1 className="text-sm font-semibold text-slate-900">{user.name}</h1>
              </div>
            </div>
            <div className="flex items-center gap-2">
              <button className="flex h-9 w-9 items-center justify-center rounded-full bg-white text-slate-700 shadow-sm ring-1 ring-slate-200">
                <Bell size={18} />
              </button>
              {user.role === 'admin' ? (
                <NavLink to="/admin" className="rounded-full border border-emerald-200 bg-emerald-50 px-3 py-1.5 text-xs font-semibold text-emerald-700">
                  Admin
                </NavLink>
              ) : null}
            </div>
          </div>
        </header>

        <main className="px-4 pb-20 pt-4">
          <div className="mx-auto max-w-md">
            <Routes>
              <Route index element={<HomePage showBalance={showBalance} setShowBalance={setShowBalance} spendableBalance={spendableBalance} approvedDepositsTotal={approvedDepositsTotal} totalRewards={totalRewards} activities={activities} />} />
              <Route path="payment" element={<PaymentPage onSubmit={() => {}} />} />
              <Route path="withdraw" element={<WithdrawPage spendableBalance={spendableBalance} onSubmit={() => {}} withdrawals={[]} />} />
              <Route path="statistics" element={<StatisticsPage spendableBalance={spendableBalance} approvedDepositsTotal={approvedDepositsTotal} totalRewards={totalRewards} withdrawals={[]} deposits={[]} />} />
              <Route path="my" element={<MyPage user={user} onLogout={onLogout} />} />
              <Route path="admin" element={<AdminPage deposits={[]} withdrawals={[]} />} />
            </Routes>
          </div>
        </main>
      </div>

      <nav className="fixed bottom-0 inset-x-0 z-30 mx-auto flex max-w-md items-center justify-between border-t border-slate-200 bg-white/90 px-2 py-2 shadow-[0_-8px_24px_rgba(15,23,42,0.08)] backdrop-blur">
        {navItems.map(({ label, to, icon: Icon }) => (
          <NavLink
            key={label}
            to={to}
            className={({ isActive }) =>
              `flex min-w-0 flex-1 flex-col items-center justify-center gap-1 rounded-2xl px-2 py-2 text-[11px] font-medium ${
                isActive ? 'text-emerald-700' : 'text-slate-500'
              }`
            }
          >
            <Icon size={18} className={location === to ? 'text-emerald-600' : ''} />
            <span>{label}</span>
          </NavLink>
        ))}
      </nav>
    </>
  );
}

function HomePage({ showBalance, setShowBalance, spendableBalance, approvedDepositsTotal, totalRewards, activities }: {
  showBalance: boolean;
  setShowBalance: (value: boolean) => void;
  spendableBalance: number;
  approvedDepositsTotal: number;
  totalRewards: number;
  activities: TransactionActivity[];
}) {
  return (
    <div className="space-y-4 pb-4">
      <section className="fintech-card overflow-hidden bg-gradient-to-br from-emerald-600 via-emerald-500 to-teal-600 p-5 text-white">
        <div className="flex items-center justify-between text-sm text-emerald-100">
          <span>Available Balance</span>
          <button onClick={() => setShowBalance(!showBalance)} className="rounded-full bg-white/15 p-2">
            {showBalance ? <EyeOff size={16} /> : <Eye size={16} />}
          </button>
        </div>
        <div className="mt-4 text-3xl font-bold money">
          {showBalance ? formatCurrency(spendableBalance) : '₹ •••••'}
        </div>
        <div className="mt-4 flex gap-2">
          <button className="flex flex-1 items-center justify-center gap-2 rounded-2xl bg-white/15 px-3 py-3 text-sm font-semibold backdrop-blur-sm">
            <ArrowDownLeft size={18} /> Deposit
          </button>
          <button className="flex flex-1 items-center justify-center gap-2 rounded-2xl bg-white/15 px-3 py-3 text-sm font-semibold backdrop-blur-sm">
            <ArrowUpRight size={18} /> Withdraw
          </button>
        </div>
      </section>

      <section className="rounded-3xl border border-emerald-200 bg-gradient-to-r from-emerald-50 to-lime-50 p-4 text-slate-900 shadow-soft">
        <div className="flex items-center justify-between">
          <div className="flex items-center gap-2">
            <Percent className="text-emerald-600" size={18} />
            <span className="text-sm font-semibold">Cashback 4.2% extra</span>
          </div>
          <span className="rounded-full bg-white px-2 py-1 text-[10px] font-semibold text-emerald-700">Live</span>
        </div>
        <p className="mt-2 text-sm text-slate-600">Earn extra on every approved deposit with MintPay rewards.</p>
      </section>

      <section className="grid grid-cols-2 gap-3">
        <MetricCard label="Today’s Income" value={formatCurrency(1860)} icon={<Wallet size={18} />} />
        <MetricCard label="Approved Deposits" value={formatCurrency(approvedDepositsTotal)} icon={<ReceiptText size={18} />} />
        <MetricCard label="Total Withdrawals" value={formatCurrency(12800)} icon={<Banknote size={18} />} />
        <MetricCard label="Total Rewards" value={formatCurrency(totalRewards)} icon={<TrendingUp size={18} />} />
      </section>

      <section className="fintech-card p-4">
        <div className="mb-3 flex items-center justify-between">
          <h3 className="text-base font-semibold text-slate-900">Recent activity</h3>
          <button className="text-xs font-medium text-emerald-700">View all</button>
        </div>
        <div className="space-y-3">
          {activities.map((item) => (
            <div key={item.id} className="flex items-start justify-between gap-3 rounded-2xl bg-slate-50 p-3">
              <div className="flex items-start gap-3">
                <div className={`mt-0.5 rounded-xl p-2 ${item.type === 'income' ? 'bg-emerald-100 text-emerald-700' : item.type === 'expense' ? 'bg-rose-100 text-rose-700' : 'bg-amber-100 text-amber-700'}`}>
                  {item.type === 'income' ? <ArrowDownLeft size={14} /> : item.type === 'expense' ? <ArrowUpRight size={14} /> : <Plus size={14} />}
                </div>
                <div>
                  <div className="font-medium text-slate-900">{item.title}</div>
                  <div className="text-xs text-slate-500">{item.subtitle}</div>
                </div>
              </div>
              <div className="text-right">
                <div className="font-semibold text-slate-900">{item.amount}</div>
                <div className="text-[10px] text-slate-400">{item.date}</div>
              </div>
            </div>
          ))}
        </div>
      </section>
    </div>
  );
}

function PaymentPage({ onSubmit }: { onSubmit: (amount: number, utr: string) => void }) {
  const [filter, setFilter] = useState<'All' | '₹100–₹500' | '₹500–₹2,000' | '₹2,000–₹10,000'>('All');
  const [selectedOrder, setSelectedOrder] = useState<PaymentOrder | null>(null);
  const [utr, setUtr] = useState('');
  const [proofName, setProofName] = useState('');

  const filteredOrders = filter === 'All' ? paymentOrders : paymentOrders.filter((order) => order.label === filter);

  const submitOrder = () => {
    if (!selectedOrder || !utr.trim()) {
      toast.error('Please add a valid UTR number');
      return;
    }

    onSubmit(selectedOrder.amount, utr);
    setSelectedOrder(null);
    setProofName('');
    setUtr('');
  };

  return (
    <div className="space-y-4 pb-4">
      <section className="rounded-3xl bg-gradient-to-br from-slate-900 via-slate-800 to-emerald-900 p-5 text-white shadow-soft">
        <div className="flex items-center justify-between text-sm text-slate-200">
          <span>Cashback 4.2%</span>
          <span className="rounded-full bg-white/10 px-2 py-1 text-[10px] font-medium">Reward active</span>
        </div>
        <div className="mt-4 flex items-center justify-between">
          <div>
            <p className="text-xs text-slate-300">Available balance</p>
            <p className="money mt-1 text-2xl font-bold">₹58,320</p>
          </div>
          <div className="rounded-2xl bg-white/10 px-3 py-2 text-right">
            <div className="text-[10px] text-slate-300">Reward earned</div>
            <div className="money text-lg font-bold">₹1,240</div>
          </div>
        </div>
      </section>

      <section className="fintech-card p-3">
        <div className="flex gap-2 overflow-x-auto pb-1">
          {(['All', '₹100–₹500', '₹500–₹2,000', '₹2,000–₹10,000'] as const).map((tab) => (
            <button
              key={tab}
              onClick={() => setFilter(tab)}
              className={`tab-pill whitespace-nowrap ${filter === tab ? 'bg-emerald-600 text-white' : 'bg-slate-100 text-slate-600'}`}
            >
              {tab}
            </button>
          ))}
        </div>
      </section>

      <section className="space-y-3">
        {filteredOrders.map((order) => (
          <div key={order.id} className="fintech-card flex items-center justify-between gap-3 p-4">
            <div>
              <div className="money text-xl font-bold text-slate-900">₹{order.amount.toLocaleString('en-IN')}</div>
              <div className="mt-1 text-xs text-slate-500">Reward: ₹{order.reward.toFixed(2)} • 4.2%</div>
            </div>
            <button
              onClick={() => setSelectedOrder(order)}
              className="rounded-full bg-emerald-600 px-4 py-2 text-sm font-semibold text-white shadow-sm"
            >
              Claim / Pay
            </button>
          </div>
        ))}
      </section>

      {selectedOrder && (
        <div className="fixed inset-0 z-40 flex items-end bg-slate-900/40 p-4 sm:items-center">
          <div className="w-full max-w-md rounded-3xl bg-white p-4 shadow-2xl">
            <div className="flex items-center justify-between">
              <h3 className="text-lg font-semibold text-slate-900">Complete payment</h3>
              <button onClick={() => setSelectedOrder(null)} className="text-slate-500">Close</button>
            </div>
            <div className="mt-4 rounded-2xl border border-dashed border-emerald-200 bg-emerald-50 p-4 text-center">
              <div className="flex items-center justify-center gap-2 text-emerald-700">
                <Smartphone size={18} />
                <span className="font-semibold">UPI: mintpay@upi</span>
              </div>
              <div className="mt-3 flex items-center justify-center gap-2">
                <button className="rounded-full bg-white px-3 py-1.5 text-xs font-medium text-slate-700 shadow-sm ring-1 ring-slate-200">
                  <Copy size={12} className="mr-1 inline" /> Copy UPI ID
                </button>
              </div>
              <div className="mt-4 flex h-24 items-center justify-center rounded-2xl bg-white text-sm text-slate-400 ring-1 ring-slate-200">
                QR code placeholder
              </div>
            </div>

            <div className="mt-4 space-y-3">
              <label className="block">
                <span className="mb-1 block text-sm font-medium text-slate-700">UTR / Reference Number</span>
                <input
                  value={utr}
                  onChange={(e) => setUtr(e.target.value)}
                  placeholder="12-digit UTR"
                  className="w-full rounded-2xl border border-slate-200 bg-slate-50 px-3 py-2.5 outline-none ring-0 transition focus:border-emerald-400"
                />
              </label>
              <label className="block">
                <span className="mb-1 block text-sm font-medium text-slate-700">Screenshot proof</span>
                <label className="flex h-12 cursor-pointer items-center justify-center gap-2 rounded-2xl border border-dashed border-slate-300 bg-slate-50 text-sm text-slate-600">
                  <FileImage size={16} />
                  <span>{proofName || 'Upload payment proof'}</span>
                  <input
                    type="file"
                    accept="image/*"
                    className="hidden"
                    onChange={(e) => setProofName(e.target.files?.[0]?.name || '')}
                  />
                </label>
              </label>
            </div>

            <button onClick={submitOrder} className="mt-5 w-full rounded-2xl bg-emerald-600 py-3 text-base font-semibold text-white">
              Submit Pending Deposit
            </button>
          </div>
        </div>
      )}
    </div>
  );
}

function WithdrawPage({ spendableBalance, onSubmit, withdrawals }: { spendableBalance: number; onSubmit: (amount: number, mode: 'UPI ID' | 'Bank Account', details: string) => void; withdrawals: WithdrawalItem[]; }) {
  const [amount, setAmount] = useState('');
  const [mode, setMode] = useState<'UPI ID' | 'Bank Account'>('UPI ID');
  const [details, setDetails] = useState('');

  const submitWithdrawal = () => {
    const value = Number(amount);
    if (mode === 'UPI ID' && !details.trim()) {
      toast.error('Add your UPI ID');
      return;
    }
    if (mode === 'Bank Account' && !details.trim()) {
      toast.error('Add account details');
      return;
    }
    onSubmit(value, mode, details);
    setAmount('');
    setDetails('');
  };

  return (
    <div className="space-y-4 pb-4">
      <section className="fintech-card p-4">
        <div className="flex items-center justify-between">
          <div>
            <p className="text-xs text-slate-500">Spendable balance</p>
            <div className="money mt-1 text-2xl font-bold text-slate-900">₹{spendableBalance.toLocaleString('en-IN')}</div>
          </div>
          <div className="rounded-2xl bg-emerald-50 p-2 text-emerald-700">
            <Landmark size={18} />
          </div>
        </div>
      </section>

      <section className="fintech-card p-4">
        <h3 className="text-base font-semibold text-slate-900">Request withdrawal</h3>
        <div className="mt-4 space-y-3">
          <label>
            <span className="mb-1 block text-sm font-medium text-slate-700">Amount</span>
            <input
              type="number"
              value={amount}
              onChange={(e) => setAmount(e.target.value)}
              placeholder="₹100 minimum"
              className="w-full rounded-2xl border border-slate-200 bg-slate-50 px-3 py-2.5 outline-none focus:border-emerald-400"
            />
          </label>

          <label>
            <span className="mb-1 block text-sm font-medium text-slate-700">Payout mode</span>
            <select
              value={mode}
              onChange={(e) => setMode(e.target.value as 'UPI ID' | 'Bank Account')}
              className="w-full rounded-2xl border border-slate-200 bg-slate-50 px-3 py-2.5 outline-none focus:border-emerald-400"
            >
              <option value="UPI ID">UPI ID</option>
              <option value="Bank Account">Bank Account</option>
            </select>
          </label>

          <label>
            <span className="mb-1 block text-sm font-medium text-slate-700">
              {mode === 'UPI ID' ? 'UPI ID' : 'Bank Account: Account No, IFSC, Holder Name'}
            </span>
            <input
              value={details}
              onChange={(e) => setDetails(e.target.value)}
              placeholder={mode === 'UPI ID' ? 'name@upi' : 'A/c No • IFSC • Holder Name'}
              className="w-full rounded-2xl border border-slate-200 bg-slate-50 px-3 py-2.5 outline-none focus:border-emerald-400"
            />
          </label>

          <button onClick={submitWithdrawal} className="w-full rounded-2xl bg-emerald-600 py-3 text-base font-semibold text-white">
            Submit withdrawal
          </button>
        </div>
      </section>

      <section className="fintech-card p-4">
        <div className="mb-3 flex items-center justify-between">
          <h3 className="text-base font-semibold text-slate-900">Withdrawal history</h3>
          <span className="text-xs font-medium text-slate-500">{withdrawals.length} records</span>
        </div>
        <div className="space-y-3">
          {withdrawals.slice(0, 3).map((item) => (
            <div key={item.id} className="rounded-2xl bg-slate-50 p-3">
              <div className="flex items-center justify-between">
                <div className="money text-lg font-bold text-slate-900">₹{item.amount.toLocaleString('en-IN')}</div>
                <span className={`rounded-full px-2 py-1 text-[10px] font-semibold ${item.status === 'Approved' ? 'bg-emerald-100 text-emerald-700' : item.status === 'Pending' ? 'bg-amber-100 text-amber-700' : 'bg-rose-100 text-rose-700'}`}>
                  {item.status}
                </span>
              </div>
              <div className="mt-2 text-xs text-slate-500">{item.mode} • {item.details}</div>
              <div className="mt-1 text-[10px] text-slate-400">{item.date}</div>
            </div>
          ))}
        </div>
      </section>
    </div>
  );
}

function StatisticsPage({ spendableBalance, approvedDepositsTotal, totalRewards, withdrawals, deposits }: { spendableBalance: number; approvedDepositsTotal: number; totalRewards: number; withdrawals: WithdrawalItem[]; deposits: DepositItem[]; }) {
  const totalApprovedWithdrawals = withdrawals
    .filter((entry) => entry.status === 'Approved')
    .reduce((sum, item) => sum + item.amount, 0);
  const approvalRate = deposits.length ? Math.round((deposits.filter((item) => item.status === 'Approved').length / deposits.length) * 100) : 0;

  return (
    <div className="space-y-4 pb-4">
      <section className="grid grid-cols-2 gap-3">
        <MetricCard label="Balance" value={formatCurrency(spendableBalance)} icon={<Wallet size={18} />} />
        <MetricCard label="Total Deposits" value={formatCurrency(approvedDepositsTotal)} icon={<ReceiptText size={18} />} />
        <MetricCard label="Total Withdrawals" value={formatCurrency(totalApprovedWithdrawals)} icon={<Banknote size={18} />} />
        <MetricCard label="4.2% Rewards" value={formatCurrency(totalRewards)} icon={<Percent size={18} />} />
      </section>

      <section className="fintech-card p-4">
        <h3 className="text-base font-semibold text-slate-900">Overview</h3>
        <div className="mt-4 grid grid-cols-2 gap-3">
          <div className="rounded-2xl bg-emerald-50 p-3">
            <div className="text-xs text-slate-500">Approved</div>
            <div className="money mt-1 text-xl font-bold text-emerald-700">{deposits.filter((item) => item.status === 'Approved').length}</div>
          </div>
          <div className="rounded-2xl bg-amber-50 p-3">
            <div className="text-xs text-slate-500">Pending</div>
            <div className="money mt-1 text-xl font-bold text-amber-700">{deposits.filter((item) => item.status === 'Pending').length}</div>
          </div>
        </div>
        <div className="mt-4">
          <div className="mb-2 flex items-center justify-between text-xs text-slate-500">
            <span>Approval rate</span>
            <span>{approvalRate}%</span>
          </div>
          <div className="h-2.5 rounded-full bg-slate-200">
            <div className="h-full rounded-full bg-emerald-500" style={{ width: `${approvalRate}%` }} />
          </div>
        </div>
      </section>
    </div>
  );
}

function MyPage({ user, onLogout }: { user: UserProfile; onLogout: () => void }) {
  const links = [
    { label: 'Transaction History', icon: ReceiptText },
    { label: 'Bank / UPI Settings', icon: Landmark },
    { label: 'Security', icon: ShieldCheck },
    { label: 'Support', icon: HelpCircle },
  ];

  return (
    <div className="space-y-4 pb-4">
      <section className="fintech-card p-4">
        <div className="flex items-center gap-3">
          <div className="flex h-14 w-14 items-center justify-center rounded-2xl bg-emerald-100 text-emerald-700">
            <UserCircle2 size={32} />
          </div>
          <div>
            <div className="text-lg font-semibold text-slate-900">{user.name}</div>
            <div className="text-sm text-slate-500">{user.phone}</div>
          </div>
        </div>
        <div className="mt-4 rounded-2xl bg-slate-50 p-3 text-sm text-slate-600">
          MintPay ID: <span className="font-semibold text-slate-900">{user.id}</span>
        </div>
      </section>

      <section className="fintech-card p-3">
        {links.map(({ label, icon: Icon }) => (
          <button key={label} className="flex w-full items-center justify-between rounded-2xl px-3 py-3 text-left text-slate-700 hover:bg-slate-50">
            <div className="flex items-center gap-3">
              <div className="rounded-xl bg-emerald-50 p-2 text-emerald-700"><Icon size={16} /></div>
              <span className="font-medium">{label}</span>
            </div>
            <ChevronRight size={16} className="text-slate-400" />
          </button>
        ))}
      </section>

      <button onClick={onLogout} className="flex w-full items-center justify-center gap-2 rounded-2xl border border-rose-200 bg-rose-50 px-4 py-3 text-base font-semibold text-rose-700">
        <LogOut size={18} /> Logout
      </button>
    </div>
  );
}

function AdminPage({ deposits, withdrawals }: { deposits: DepositItem[]; withdrawals: WithdrawalItem[] }) {
  return (
    <div className="space-y-4 pb-4">
      <section className="grid grid-cols-2 gap-3">
        <MetricCard label="Users" value="1,248" icon={<UserCircle2 size={18} />} />
        <MetricCard label="Deposit volume" value={'₹1.8L'} icon={<ReceiptText size={18} />} />
        <MetricCard label="Pending deposits" value={String(deposits.filter((item) => item.status === 'Pending').length)} icon={<ClockIcon />} />
        <MetricCard label="Pending payouts" value={String(withdrawals.filter((item) => item.status === 'Pending').length)} icon={<Wallet size={18} />} />
      </section>

      <section className="fintech-card p-4">
        <h3 className="text-base font-semibold text-slate-900">Deposit requests</h3>
        <div className="mt-3 space-y-3">
          {deposits.slice(0, 2).map((item) => (
            <div key={item.id} className="rounded-2xl bg-slate-50 p-3">
              <div className="flex items-center justify-between">
                <div>
                  <div className="font-semibold text-slate-900">{item.id}</div>
                  <div className="text-xs text-slate-500">UTR: {item.utr}</div>
                </div>
                <span className="rounded-full bg-amber-100 px-2 py-1 text-[10px] font-semibold text-amber-700">{item.status}</span>
              </div>
              <div className="mt-2 flex items-center justify-between text-sm">
                <span className="text-slate-500">Amount</span>
                <span className="money font-semibold">₹{item.amount.toLocaleString('en-IN')}</span>
              </div>
              <div className="mt-1 flex items-center justify-between text-sm">
                <span className="text-slate-500">Reward</span>
                <span className="money font-semibold text-emerald-700">₹{item.reward.toFixed(2)}</span>
              </div>
              <div className="mt-3 flex gap-2">
                <button className="flex-1 rounded-xl bg-emerald-600 px-3 py-2 text-sm font-semibold text-white">Approve</button>
                <button className="flex-1 rounded-xl bg-rose-50 px-3 py-2 text-sm font-semibold text-rose-700">Reject</button>
              </div>
            </div>
          ))}
        </div>
      </section>
    </div>
  );
}

function AuthPage({ onLogin }: { onLogin: (name: string, phone: string, password: string, isSignup: boolean) => void }) {
  const [isSignup, setIsSignup] = useState(false);
  const [name, setName] = useState('');
  const [phone, setPhone] = useState('');
  const [password, setPassword] = useState('');
  const [inviteCode, setInviteCode] = useState('');

  const handleSubmit = (event: FormEvent) => {
    event.preventDefault();
    if (!name.trim() || !phone.trim() || !password.trim()) {
      toast.error('Please fill in all required fields');
      return;
    }
    onLogin(name.trim(), phone.trim(), password.trim(), isSignup);
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-emerald-600 via-emerald-500 to-teal-600 px-4 py-8 text-white">
      <div className="mx-auto max-w-md">
        <div className="flex items-center justify-center gap-3">
          <div className="flex h-14 w-14 items-center justify-center rounded-2xl bg-white/15 backdrop-blur-sm">
            <LeafIcon />
          </div>
          <div>
            <div className="text-2xl font-bold">MintPay</div>
            <div className="text-xs text-emerald-50/80">Smart wallet & cashback</div>
          </div>
        </div>

        <div className="mt-8 rounded-[2rem] bg-white p-5 text-slate-900 shadow-2xl">
          <div className="mb-5 flex rounded-2xl bg-slate-100 p-1">
            <button
              onClick={() => setIsSignup(false)}
              className={`flex-1 rounded-xl px-3 py-2 text-sm font-semibold ${!isSignup ? 'bg-white text-emerald-700 shadow-sm' : 'text-slate-500'}`}
            >
              Login
            </button>
            <button
              onClick={() => setIsSignup(true)}
              className={`flex-1 rounded-xl px-3 py-2 text-sm font-semibold ${isSignup ? 'bg-white text-emerald-700 shadow-sm' : 'text-slate-500'}`}
            >
              Sign up
            </button>
          </div>

          <form onSubmit={handleSubmit} className="space-y-3">
            {isSignup && (
              <label className="block">
                <span className="mb-1 block text-sm font-medium text-slate-700">User Name</span>
                <input value={name} onChange={(e) => setName(e.target.value)} placeholder="Enter full name" className="w-full rounded-2xl border border-slate-200 bg-slate-50 px-3 py-2.5 outline-none focus:border-emerald-400" />
              </label>
            )}

            {!isSignup && (
              <label className="block">
                <span className="mb-1 block text-sm font-medium text-slate-700">Phone Number</span>
                <input value={phone} onChange={(e) => setPhone(e.target.value)} placeholder="+91 98765 43210" className="w-full rounded-2xl border border-slate-200 bg-slate-50 px-3 py-2.5 outline-none focus:border-emerald-400" />
              </label>
            )}

            {isSignup && (
              <label className="block">
                <span className="mb-1 block text-sm font-medium text-slate-700">Phone Number (+91)</span>
                <input value={phone} onChange={(e) => setPhone(e.target.value)} placeholder="98765 43210" className="w-full rounded-2xl border border-slate-200 bg-slate-50 px-3 py-2.5 outline-none focus:border-emerald-400" />
              </label>
            )}

            <label className="block">
              <span className="mb-1 block text-sm font-medium text-slate-700">Password</span>
              <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Enter password" className="w-full rounded-2xl border border-slate-200 bg-slate-50 px-3 py-2.5 outline-none focus:border-emerald-400" />
            </label>

            {isSignup && (
              <label className="block">
                <span className="mb-1 block text-sm font-medium text-slate-700">Invite Code (optional)</span>
                <input value={inviteCode} onChange={(e) => setInviteCode(e.target.value)} placeholder="Optional" className="w-full rounded-2xl border border-slate-200 bg-slate-50 px-3 py-2.5 outline-none focus:border-emerald-400" />
              </label>
            )}

            {!isSignup && (
              <div className="flex items-center justify-between text-sm text-slate-600">
                <label className="flex items-center gap-2">
                  <input type="checkbox" className="h-4 w-4 accent-emerald-600" defaultChecked />
                  Remember Me
                </label>
                <button type="button" className="text-emerald-700">Need help?</button>
              </div>
            )}

            <button type="submit" className="mt-3 w-full rounded-2xl bg-emerald-600 py-3 text-base font-semibold text-white">
              {isSignup ? 'Create account' : 'Login'}
            </button>
          </form>

          <div className="mt-4 flex items-center gap-2 rounded-2xl border border-emerald-100 bg-emerald-50 p-3 text-sm text-emerald-800">
            <HelpCircle size={18} />
            AI assistant: Try resetting your password or checking your UTR status.
          </div>
        </div>
      </div>
    </div>
  );
}

function MetricCard({ label, value, icon }: { label: string; value: string; icon: React.ReactNode }) {
  return (
    <div className="fintech-card p-4">
      <div className="mb-3 flex h-10 w-10 items-center justify-center rounded-2xl bg-emerald-50 text-emerald-700">{icon}</div>
      <div className="text-xs text-slate-500">{label}</div>
      <div className="money mt-1 text-xl font-bold text-slate-900">{value}</div>
    </div>
  );
}

function ClockIcon() {
  return <div className="h-4 w-4 rounded-full border-2 border-current" />;
}

function LeafIcon() {
  return <div className="h-5 w-5 rounded-full bg-white/20" />;
}

export default App;
