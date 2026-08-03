[app_cobrancas_mobile_ultimate (2).tsx](https://github.com/user-attachments/files/30644547/app_cobrancas_mobile_ultimate.2.tsx)
import React, { useState, useEffect, useMemo } from 'react';
import { 
  Home, 
  Calendar as CalendarIcon, 
  Plus, 
  Trophy, 
  Settings, 
  Moon, 
  Sun, 
  ChevronRight, 
  AlertCircle, 
  CheckCircle,
  MessageCircle,
  ChevronLeft,
  X,
  CreditCard,
  User,
  Phone,
  FileText,
  Edit3,
  Trash2,
  Clock,
  Send,
  TrendingUp,
  TrendingDown,
  Award,
  Save,
  Lock,
  Percent,
  Bell,
  Download,
  Shield,
  Filter,
  Paperclip,
  Eye,
  EyeOff,
  Sparkles,
  DollarSign
} from 'lucide-react';

// --- FUNÇÕES UTILITÁRIAS ---
const formatCurrency = (value) => {
  return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(value);
};

const formatDateBr = (dateStr) => {
  if (!dateStr) return '';
  const [year, month, day] = dateStr.split('-');
  return `${day}/${month}/${year}`;
};

const getStatus = (dueDateStr, isPaid, remainingAmount, originalAmount) => {
  if (isPaid || remainingAmount <= 0) return 'paid';
  const today = new Date();
  today.setHours(0, 0, 0, 0);
  const [year, month, day] = dueDateStr.split('-');
  const dueDate = new Date(year, month - 1, day);
  return dueDate < today ? 'overdue' : 'pending';
};

const NavItem = ({ icon, label, isActive, onClick }) => (
  <button 
    onClick={onClick}
    className={`flex flex-col items-center justify-center w-12 gap-1 transition-colors
      ${isActive ? 'text-indigo-600 dark:text-indigo-400' : 'text-slate-400 dark:text-slate-500 hover:text-slate-600 dark:hover:text-slate-300'}
    `}
  >
    {React.cloneElement(icon, { size: 22, strokeWidth: isActive ? 2.5 : 2 })}
    <span className="text-[10px] font-medium">{label}</span>
  </button>
);

export default function App() {
  // --- ESTADOS GLOBAIS ---
  const [darkMode, setDarkMode] = useState(true);
  const [activeTab, setActiveTab] = useState('home');
  const [userName, setUserName] = useState('Marcos');
  const [tempUserName, setTempUserName] = useState('Marcos');
  
  // Segurança / PIN / Vault
  const [usePin, setUsePin] = useState(false);
  const [pinCode, setPinCode] = useState('1234');
  const [isLocked, setIsLocked] = useState(false);
  const [enteredPin, setEnteredPin] = useState('');
  const [vaultMode, setVaultMode] = useState(false);

  // Ajustes
  const [useInterest, setUseInterest] = useState(false);
  const [interestRate, setInterestRate] = useState('2.0');
  const [notificationsEnabled, setNotificationsEnabled] = useState(true);
  const [pixKey, setPixKey] = useState('11999887766');

  // Modais e Estados operacionais
  const [isAddModalOpen, setIsAddModalOpen] = useState(false);
  const [selectedDebt, setSelectedDebt] = useState(null);
  const [isEditing, setIsEditing] = useState(false);
  const [newLogText, setNewLogText] = useState('');
  const [debtToDelete, setDebtToDelete] = useState(null);
  const [filterStatus, setFilterStatus] = useState('all');

  // Estados de Amortização e Provas na Ficha
  const [partialAmount, setPartialAmount] = useState('');
  const [selectedTone, setSelectedTone] = useState('friendly');
  
  // Lista inicial rica em dados
  const [debts, setDebts] = useState([
    { 
      id: 1, name: 'Carlos (Cunhado)', amount: 850.00, remaining: 850.00, dueDate: '2026-07-05', isPaid: false, type: 'Empréstimo', phone: '5511912345678', 
      logs: [{ id: 101, text: 'Disse que o pagamento sai sexta-feira.', date: '2026-07-10' }], 
      payments: [], attachments: ['https://images.unsplash.com/photo-1618005182384-a83a8bd57fbe?w=300'] 
    },
    { 
      id: 2, name: 'Ana Souza', amount: 300.00, remaining: 150.00, dueDate: '2026-08-12', isPaid: false, type: 'Racha do Jantar', phone: '5511912345678', 
      logs: [], payments: [{ id: 201, amount: 150.00, date: '2026-08-01' }], attachments: [] 
    },
    { 
      id: 3, name: 'Felipe', amount: 1500.00, remaining: 1500.00, dueDate: '2026-08-15', isPaid: false, type: 'Conserto Carro', phone: '', 
      logs: [], payments: [], attachments: [] 
    },
    { 
      id: 4, name: 'João Silva', amount: 400.00, remaining: 0, dueDate: '2026-07-20', isPaid: true, type: 'Conta de Luz', phone: '', 
      logs: [], payments: [{ id: 401, amount: 400.00, date: '2026-07-19' }], attachments: [] 
    },
  ]);

  const computedDebts = useMemo(() => {
    const rate = parseFloat(interestRate) || 0;
    const today = new Date();

    return debts.map(debt => {
      let finalRemaining = debt.remaining;
      const status = getStatus(debt.dueDate, debt.isPaid, debt.remaining, debt.amount);

      if (useInterest && status === 'overdue' && !debt.isPaid && debt.remaining > 0) {
        const [y, m, d] = debt.dueDate.split('-');
        const due = new Date(y, m - 1, d);
        const diffDays = Math.ceil(Math.abs(today - due) / (1000 * 60 * 60 * 24));
        const penalty = debt.amount * (rate / 100) * (diffDays / 30);
        finalRemaining += penalty;
      }

      return {
        ...debt,
        calculatedRemaining: finalRemaining,
        status
      };
    }).sort((a, b) => new Date(a.dueDate) - new Date(b.dueDate));
  }, [debts, useInterest, interestRate]);

  const summary = useMemo(() => {
    let totalReceivable = 0; let overdue = 0; let receivedThisMonth = 0; let activeCount = 0;
    const currentMonth = new Date().getMonth() + 1;
    const currentYear = new Date().getFullYear();

    computedDebts.forEach(debt => {
      const [y, m] = debt.dueDate.split('-');
      const isThisMonth = parseInt(m) === currentMonth && parseInt(y) === currentYear;
      
      if (debt.payments && debt.payments.length > 0) {
        debt.payments.forEach(p => {
          const [py, pm] = p.date.split('-');
          if (parseInt(pm) === currentMonth && parseInt(py) === currentYear) {
            receivedThisMonth += p.amount;
          }
        });
      }

      if (!debt.isPaid && debt.remaining > 0) {
        totalReceivable += debt.calculatedRemaining;
        activeCount++;
        if (debt.status === 'overdue') overdue += debt.calculatedRemaining;
      }
    });
    return { totalReceivable, overdue, receivedThisMonth, activeCount };
  }, [computedDebts]);

  useEffect(() => {
    if (darkMode) document.documentElement.classList.add('dark');
    else document.documentElement.classList.remove('dark');
  }, [darkMode]);

  const handlePinSubmit = (e) => {
    e.preventDefault();
    if (enteredPin === pinCode) {
      setIsLocked(false);
      setVaultMode(false);
      setEnteredPin('');
    } else if (enteredPin === '0000') {
      setIsLocked(false);
      setVaultMode(true);
      setEnteredPin('');
    } else {
      alert("PIN incorreto! (Dica: tente 1234 ou 0000 para Modo Disfarce)");
      setEnteredPin('');
    }
  };

  const markAsPaidCompletely = (id, e) => {
    if(e) e.stopPropagation();
    const todayStr = new Date().toISOString().split('T')[0];
    setDebts(debts.map(d => {
      if (d.id === id) {
        const paidAmount = d.remaining;
        return {
          ...d,
          remaining: 0,
          isPaid: true,
          payments: [...d.payments, { id: Date.now(), amount: paidAmount, date: todayStr }]
        };
      }
      return d;
    }));
    if(selectedDebt && selectedDebt.id === id) {
      setSelectedDebt(prev => ({ ...prev, remaining: 0, isPaid: true, payments: [...prev.payments, { id: Date.now(), amount: prev.remaining, date: todayStr }] }));
    }
  };

  const handlePartialPayment = (e) => {
    e.preventDefault();
    const val = parseFloat(partialAmount);
    if (!val || val <= 0 || val > selectedDebt.remaining) {
      alert("Valor de abatimento inválido.");
      return;
    }

    const todayStr = new Date().toISOString().split('T')[0];
    const newRemaining = selectedDebt.remaining - val;
    const isNowPaid = newRemaining <= 0;
    const newPayment = { id: Date.now(), amount: val, date: todayStr };

    setDebts(debts.map(d => {
      if (d.id === selectedDebt.id) {
        const updated = {
          ...d,
          remaining: newRemaining,
          isPaid: isNowPaid,
          payments: [...d.payments, newPayment]
        };
        setSelectedDebt(updated);
        return updated;
      }
      return d;
    }));
    setPartialAmount('');
  };

  const confirmDelete = () => {
    setDebts(debts.filter(d => d.id !== debtToDelete));
    setSelectedDebt(null);
    setDebtToDelete(null);
  };

  const handleAddLog = (e) => {
    e.preventDefault();
    if(!newLogText.trim()) return;
    const newLog = { id: Date.now(), text: newLogText, date: new Date().toISOString().split('T')[0] };
    setDebts(debts.map(d => {
      if(d.id === selectedDebt.id) {
        const updatedDebt = { ...d, logs: [newLog, ...(d.logs || [])] };
        setSelectedDebt(updatedDebt);
        return updatedDebt;
      }
      return d;
    }));
    setNewLogText('');
  };

  const handleSaveDebt = (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const amountVal = parseFloat(formData.get('amount'));
    const debtData = {
      name: formData.get('name'),
      amount: amountVal,
      remaining: amountVal,
      dueDate: formData.get('dueDate'),
      type: formData.get('type'),
      phone: formData.get('phone'),
    };

    if (isEditing && selectedDebt) {
      const updatedDebt = { ...selectedDebt, ...debtData, remaining: Math.min(debtData.amount, selectedDebt.remaining) };
      setDebts(debts.map(d => d.id === selectedDebt.id ? updatedDebt : d));
      setSelectedDebt(updatedDebt);
      setIsEditing(false);
    } else {
      const newDebt = { ...debtData, id: Date.now(), isPaid: false, logs: [], payments: [], attachments: [] };
      setDebts([...debts, newDebt]);
      setIsAddModalOpen(false);
      setActiveTab('home');
    }
  };

  // Correção robusta para abertura do WhatsApp
  const handleWhatsApp = (debt, e) => {
    if(e) e.stopPropagation();
    
    // Remove qualquer caractere que não seja número (parênteses, traços, espaços, etc.)
    const cleanPhone = (debt.phone || '').replace(/\D/g, '');
    
    if (!cleanPhone) {
      alert("Número de WhatsApp não cadastrado ou inválido para este contato.");
      return;
    }

    let message = "";
    const valStr = formatCurrency(debt.calculatedRemaining);

    if (selectedTone === 'friendly') {
      message = `Fala ${debt.name}, blza? Passando para lembrar daquele valor de ${valStr} ref. a "${debt.type}" que venceu dia ${debt.dueDate.split('-')[2]}. Consegue me dar um salve com o PIX (${pixKey}) quando puder? Abraço!`;
    } else if (selectedTone === 'formal') {
      message = `Prezado(a) ${debt.name}, venho por meio desta mensagem solicitar a regularização do saldo pendente no valor de ${valStr} referente a "${debt.type}". Chave PIX para transferência: ${pixKey}. Obrigado.`;
    } else {
      message = `E aí ${debt.name}! O boleto da amizade (${valStr} - ${debt.type}) tá batendo na porta! 🚨 Manda o PIX (${pixKey}) para o papai ficar feliz! Hahaha`;
    }

    const url = `https://wa.me/${cleanPhone}?text=${encodeURIComponent(message)}`;
    window.open(url, '_blank', 'noopener,noreferrer');
  };

  const handleFileUploadSim = () => {
    const sampleImage = "https://images.unsplash.com/photo-1554224155-8d04cb21cd6c?w=300";
    setDebts(debts.map(d => {
      if (d.id === selectedDebt.id) {
        const updated = { ...d, attachments: [...d.attachments, sampleImage] };
        setSelectedDebt(updated);
        return updated;
      }
      return d;
    }));
  };

  const exportData = () => {
    const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(debts, null, 2));
    const downloadAnchor = document.createElement('a');
    downloadAnchor.setAttribute("href", dataStr);
    downloadAnchor.setAttribute("download", `relatorio_dividas_${userName}.json`);
    document.body.appendChild(downloadAnchor);
    downloadAnchor.click();
    downloadAnchor.remove();
  };

  const DebtCard = ({ debt }) => (
    <div 
      onClick={() => { setSelectedDebt(debt); setIsEditing(false); }}
      className="bg-white dark:bg-slate-800 rounded-2xl p-4 shadow-sm border border-slate-100 dark:border-slate-700 cursor-pointer active:scale-[0.98] transition-transform"
    >
      <div className="flex items-center justify-between mb-3">
        <div className="flex items-center gap-3">
          <div className={`w-10 h-10 rounded-full flex items-center justify-center font-bold text-sm
            ${debt.status === 'overdue' ? 'bg-rose-100 text-rose-600 dark:bg-rose-500/20 dark:text-rose-400' : 
              debt.status === 'paid' ? 'bg-emerald-100 text-emerald-600 dark:bg-emerald-500/20 dark:text-emerald-400' :
              'bg-slate-100 text-slate-600 dark:bg-slate-700 dark:text-slate-300'}`}>
            {debt.name.charAt(0)}
          </div>
          <div>
            <h4 className="font-semibold text-slate-800 dark:text-slate-200">{debt.name}</h4>
            <p className="text-xs text-slate-500 dark:text-slate-400">
              {debt.status === 'overdue' ? <span className="text-rose-500 font-medium">Em atraso</span> : 
               debt.status === 'paid' ? <span className="text-emerald-500 font-medium">Quitado</span> :
               `Vence a ${debt.dueDate.split('-')[2]}`} • {debt.type}
            </p>
          </div>
        </div>
        <div className="text-right">
          <p className={`font-bold ${debt.status === 'overdue' ? 'text-rose-600 dark:text-rose-400' : 'text-slate-800 dark:text-slate-200'}`}>
            {formatCurrency(debt.calculatedRemaining)}
          </p>
          {debt.remaining < debt.amount && !debt.isPaid && (
            <span className="text-[10px] text-indigo-500 block">Parcialmente pago</span>
          )}
        </div>
      </div>
      
      {!debt.isPaid && (
        <div className="flex gap-2 mt-2 pt-2 border-t border-slate-100 dark:border-slate-700">
          <button onClick={(e) => handleWhatsApp(debt, e)} className="flex-1 py-1.5 bg-indigo-50 dark:bg-indigo-500/10 text-indigo-600 dark:text-indigo-400 rounded-lg text-xs font-semibold flex items-center justify-center gap-1 hover:bg-indigo-100 transition-colors">
            <MessageCircle size={14} /> Cobrar WhatsApp
          </button>
          <button onClick={(e) => markAsPaidCompletely(debt.id, e)} className="flex-1 py-1.5 bg-emerald-50 dark:bg-emerald-500/10 text-emerald-600 dark:text-emerald-400 rounded-lg text-xs font-semibold flex items-center justify-center gap-1 hover:bg-emerald-100 transition-colors">
            <CheckCircle size={14} /> Quitar
          </button>
        </div>
      )}
    </div>
  );

  const DashboardView = () => {
    const filteredDebts = computedDebts.filter(d => {
      if (filterStatus === 'all') return true;
      if (filterStatus === 'pending') return d.status === 'pending';
      if (filterStatus === 'overdue') return d.status === 'overdue';
      if (filterStatus === 'paid') return d.status === 'paid';
      return true;
    });

    const topDebtorsBar = computedDebts.filter(d => !d.isPaid && d.remaining > 0).slice(0, 4);
    const maxBarValue = Math.max(...topDebtorsBar.map(d => d.calculatedRemaining), 1);

    return (
      <div className="space-y-6 animate-in fade-in slide-in-from-bottom-4 duration-300 pb-24">
        {vaultMode && (
          <div className="bg-amber-500/10 border border-amber-500/30 rounded-2xl p-4 text-center">
            <p className="text-xs font-bold text-amber-600 dark:text-amber-400">⚠️ Modo Disfarce Ativo: A exibir dados simulados de cofre.</p>
          </div>
        )}

        <div className="grid grid-cols-2 gap-4">
          <div className="col-span-2 bg-gradient-to-br from-indigo-600 to-indigo-800 rounded-2xl p-5 shadow-lg text-white">
            <p className="text-indigo-200 text-sm font-medium mb-1">Total a Receber</p>
            <h2 className="text-3xl font-bold">{formatCurrency(summary.totalReceivable)}</h2>
            <div className="mt-4 flex items-center justify-between text-sm">
              <span className="flex items-center gap-1 text-indigo-100">
                <AlertCircle size={16} /> {summary.activeCount} Pessoas devendo
              </span>
            </div>
          </div>
          <div className="bg-white dark:bg-slate-800 rounded-2xl p-4 shadow-sm border border-slate-100 dark:border-slate-700">
            <div className="flex items-center gap-2 mb-2">
              <div className="p-2 bg-rose-100 dark:bg-rose-500/20 rounded-full"><TrendingDown size={16} className="text-rose-600 dark:text-rose-400" /></div>
              <p className="text-xs text-slate-500 dark:text-slate-400 font-medium">Em Atraso</p>
            </div>
            <p className="text-lg font-bold text-rose-600 dark:text-rose-400">{formatCurrency(summary.overdue)}</p>
          </div>
          <div className="bg-white dark:bg-slate-800 rounded-2xl p-4 shadow-sm border border-slate-100 dark:border-slate-700">
            <div className="flex items-center gap-2 mb-2">
              <div className="p-2 bg-emerald-100 dark:bg-emerald-500/20 rounded-full"><TrendingUp size={16} className="text-emerald-600 dark:text-emerald-400" /></div>
              <p className="text-xs text-slate-500 dark:text-slate-400 font-medium">Recebido (Mês)</p>
            </div>
            <p className="text-lg font-bold text-emerald-600 dark:text-emerald-400">{formatCurrency(summary.receivedThisMonth)}</p>
          </div>
        </div>

        {/* Gráfico de Barras Interativo */}
        <div className="bg-white dark:bg-slate-800 rounded-2xl p-5 shadow-sm border border-slate-100 dark:border-slate-700">
          <h3 className="text-sm font-bold text-slate-800 dark:text-slate-100 mb-4 flex items-center justify-between">
            <span> Top Maiores Devedores</span>
          </h3>
          <div className="space-y-3">
            {topDebtorsBar.map((item, i) => {
              const widthPercent = (item.calculatedRemaining / maxBarValue) * 100;
              return (
                <div key={i} className="space-y-1">
                  <div className="flex justify-between text-xs text-slate-600 dark:text-slate-400">
                    <span className="font-medium">{item.name}</span>
                    <span className="font-bold">{formatCurrency(item.calculatedRemaining)}</span>
                  </div>
                  <div className="h-2.5 bg-slate-100 dark:bg-slate-700 rounded-full overflow-hidden">
                    <div className="h-full bg-indigo-600 rounded-full transition-all duration-500" style={{ width: `${widthPercent}%` }}></div>
                  </div>
                </div>
              );
            })}
            {topDebtorsBar.length === 0 && (
              <p className="text-xs text-center text-slate-400 py-2">Sem pendências para exibir no gráfico.</p>
            )}
          </div>
        </div>

        <div>
          <div className="flex items-center justify-between mb-4">
            <h3 className="text-lg font-bold text-slate-800 dark:text-slate-100">Filtro de Dívidas</h3>
            <div className="flex bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl p-1 text-xs">
              <button onClick={() => setFilterStatus('all')} className={`px-2 py-1 rounded-lg font-medium ${filterStatus === 'all' ? 'bg-indigo-600 text-white' : 'text-slate-500'}`}>Todos</button>
              <button onClick={() => setFilterStatus('overdue')} className={`px-2 py-1 rounded-lg font-medium ${filterStatus === 'overdue' ? 'bg-rose-600 text-white' : 'text-slate-500'}`}>Atraso</button>
              <button onClick={() => setFilterStatus('paid')} className={`px-2 py-1 rounded-lg font-medium ${filterStatus === 'paid' ? 'bg-emerald-600 text-white' : 'text-slate-500'}`}>Quitados</button>
            </div>
          </div>
          <div className="space-y-3">
            {filteredDebts.map(debt => <DebtCard key={debt.id} debt={debt} />)}
            {filteredDebts.length === 0 && (
               <div className="text-center py-6 text-slate-500">Nenhum registo encontrado.</div>
            )}
          </div>
        </div>
      </div>
    );
  };

  const CalendarView = () => {
    const today = new Date();
    const [currentDate, setCurrentDate] = useState(new Date(today.getFullYear(), today.getMonth(), 1));
    const [selectedDateStr, setSelectedDateStr] = useState(today.toISOString().split('T')[0]);
    
    const nextMonth = () => setCurrentDate(new Date(currentDate.getFullYear(), currentDate.getMonth() + 1, 1));
    const prevMonth = () => setCurrentDate(new Date(currentDate.getFullYear(), currentDate.getMonth() - 1, 1));

    const year = currentDate.getFullYear();
    const month = currentDate.getMonth();
    const daysInMonth = new Date(year, month + 1, 0).getDate();
    const firstDayOfWeek = new Date(year, month, 1).getDay();
    const monthNames = ["Janeiro", "Fevereiro", "Março", "Abril", "Maio", "Junho", "Julho", "Agosto", "Setembro", "Outubro", "Novembro", "Dezembro"];

    const days = Array.from({ length: daysInMonth }, (_, i) => {
      const day = i + 1;
      const dateStr = `${year}-${String(month + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
      const dayDebts = computedDebts.filter(d => d.dueDate === dateStr);
      let marker = null;
      if (dayDebts.length > 0) {
        if (dayDebts.some(d => d.status === 'overdue')) marker = 'overdue';
        else if (dayDebts.some(d => d.status === 'pending')) marker = 'pending';
        else marker = 'paid';
      }
      return { day, dateStr, marker };
    });

    const blanks = Array.from({ length: firstDayOfWeek }, (_, i) => i);
    const selectedDebts = computedDebts.filter(d => d.dueDate === selectedDateStr);

    return (
      <div className="animate-in fade-in slide-in-from-right-4 duration-300 pb-24 h-full flex flex-col">
        <div className="bg-white dark:bg-slate-800 rounded-3xl p-5 shadow-sm border border-slate-100 dark:border-slate-700 mb-6">
          <div className="flex items-center justify-between mb-6">
            <button onClick={prevMonth} className="p-2 bg-slate-50 dark:bg-slate-700 rounded-full text-slate-600 dark:text-slate-300"><ChevronLeft size={20} /></button>
            <h2 className="text-lg font-bold text-slate-800 dark:text-slate-100">{monthNames[month]} {year}</h2>
            <button onClick={nextMonth} className="p-2 bg-slate-50 dark:bg-slate-700 rounded-full text-slate-600 dark:text-slate-300"><ChevronRight size={20} /></button>
          </div>
          <div className="grid grid-cols-7 gap-1 mb-2 text-center">
            {['D', 'S', 'T', 'Q', 'Q', 'S', 'S'].map((d, i) => <div key={i} className="text-xs font-medium text-slate-400">{d}</div>)}
          </div>
          <div className="grid grid-cols-7 gap-1 text-center">
            {blanks.map(b => <div key={`blank-${b}`} className="h-10"></div>)}
            {days.map((d) => {
              const isSelected = selectedDateStr === d.dateStr;
              const isToday = d.dateStr === today.toISOString().split('T')[0];
              return (
                <button 
                  key={d.day} onClick={() => setSelectedDateStr(d.dateStr)}
                  className={`relative h-10 w-full flex items-center justify-center rounded-full text-sm font-medium transition-all
                    ${isSelected ? 'bg-indigo-600 text-white shadow-md' : 'text-slate-700 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-700'}
                    ${isToday && !isSelected ? 'border border-indigo-400 text-indigo-600 dark:text-indigo-400' : ''}
                  `}
                >
                  {d.day}
                  {d.marker && <span className={`absolute bottom-1 w-1.5 h-1.5 rounded-full ${d.marker === 'overdue' ? 'bg-rose-500' : d.marker === 'pending' ? (isSelected ? 'bg-indigo-200' : 'bg-amber-400') : 'bg-emerald-500'}`}></span>}
                </button>
              )
            })}
          </div>
        </div>
        <div className="flex-1">
          <h3 className="text-sm font-bold text-slate-500 dark:text-slate-400 mb-3 uppercase tracking-wider">Agenda do Dia • {selectedDateStr.split('-').reverse().join('/')}</h3>
          {selectedDebts.length === 0 ? (
            <div className="text-center py-10 bg-slate-50 dark:bg-slate-800/50 rounded-2xl border border-dashed border-slate-200 dark:border-slate-700">
              <CalendarIcon size={40} className="mx-auto text-slate-300 dark:text-slate-600 mb-3" />
              <p className="text-slate-500 dark:text-slate-400 text-sm">Sem registos neste dia.</p>
            </div>
          ) : (
            <div className="space-y-3">{selectedDebts.map(debt => <DebtCard key={debt.id} debt={debt} />)}</div>
          )}
        </div>
      </div>
    );
  };

  const RankingView = () => {
    const stats = {};
    computedDebts.forEach(debt => {
      if (!stats[debt.name]) {
        stats[debt.name] = { name: debt.name, totalOwed: 0, totalOverdue: 0, totalPaid: 0 };
      }
      if (debt.payments && debt.payments.length > 0) {
        debt.payments.forEach(p => stats[debt.name].totalPaid += p.amount);
      }
      if (!debt.isPaid && debt.remaining > 0) {
        stats[debt.name].totalOwed += debt.calculatedRemaining;
        if (debt.status === 'overdue') stats[debt.name].totalOverdue += debt.calculatedRemaining;
      }
    });

    const people = Object.values(stats);
    const topDebtors = [...people].filter(p => p.totalOwed > 0).sort((a, b) => b.totalOwed - a.totalOwed);
    const worstDefaulter = [...people].filter(p => p.totalOverdue > 0).sort((a, b) => b.totalOverdue - a.totalOverdue)[0];
    const bestPayer = [...people].filter(p => p.totalPaid > 0).sort((a, b) => b.totalPaid - a.totalPaid)[0];

    return (
      <div className="animate-in fade-in slide-in-from-right-4 duration-300 pb-24 h-full">
        <h2 className="text-xl font-bold text-slate-800 dark:text-slate-100 mb-6 flex items-center gap-2">
          <Trophy className="text-amber-500" /> Score & Pódio de Amigos
        </h2>

        <div className="grid grid-cols-2 gap-4 mb-6">
          <div className="bg-emerald-50 dark:bg-emerald-500/10 rounded-2xl p-4 border border-emerald-100 dark:border-emerald-500/20">
            <div className="flex items-center gap-2 text-emerald-600 dark:text-emerald-400 mb-2">
              <Award size={18} />
              <p className="text-xs font-bold uppercase">Melhor Pagador</p>
            </div>
            {bestPayer ? (
              <>
                <p className="font-bold text-slate-800 dark:text-slate-200 truncate">{bestPayer.name}</p>
                <p className="text-sm text-emerald-600 dark:text-emerald-400 font-medium">Quitou {formatCurrency(bestPayer.totalPaid)}</p>
              </>
            ) : (
              <p className="text-xs text-slate-500">Sem registos.</p>
            )}
          </div>

          <div className="bg-rose-50 dark:bg-rose-500/10 rounded-2xl p-4 border border-rose-100 dark:border-rose-500/20">
            <div className="flex items-center gap-2 text-rose-600 dark:text-rose-400 mb-2">
              <TrendingDown size={18} />
              <p className="text-xs font-bold uppercase">Maior Devedor</p>
            </div>
            {worstDefaulter ? (
              <>
                <p className="font-bold text-slate-800 dark:text-slate-200 truncate">{worstDefaulter.name}</p>
                <p className="text-sm text-rose-600 dark:text-rose-400 font-medium">Atrasou {formatCurrency(worstDefaulter.totalOverdue)}</p>
              </>
            ) : (
              <p className="text-xs text-slate-500">Sem atrasos graves.</p>
            )}
          </div>
        </div>

        <h3 className="text-sm font-bold text-slate-500 dark:text-slate-400 mb-3 uppercase tracking-wider">Ranking Geral de Pendências</h3>
        {topDebtors.length === 0 ? (
          <div className="text-center py-12 text-slate-500 bg-white dark:bg-slate-800 rounded-2xl border border-slate-100 dark:border-slate-700">
            Ninguém com dívidas ativas! 🎉
          </div>
        ) : (
          <div className="space-y-3">
            {topDebtors.map((person, index) => (
              <div key={index} className={`bg-white dark:bg-slate-800 rounded-2xl p-4 shadow-sm flex items-center relative overflow-hidden border ${index === 0 ? 'border-amber-400 dark:border-amber-600/50' : 'border-slate-100 dark:border-slate-700'}`}>
                {index === 0 && <div className="absolute top-0 left-0 w-1.5 h-full bg-amber-400"></div>}
                <div className="w-10 text-center font-bold text-2xl text-slate-300 dark:text-slate-600">#{index + 1}</div>
                <div className="flex-1 ml-2">
                  <h4 className="font-bold text-slate-800 dark:text-slate-200">{person.name}</h4>
                  <span className="text-xs text-slate-400">Score de pontualidade: {person.totalOverdue > 0 ? 'Baixo ⚠️' : 'Excelente ⭐'}</span>
                </div>
                <div className="text-right">
                  <p className="font-bold text-lg text-slate-800 dark:text-slate-200">{formatCurrency(person.totalOwed)}</p>
                </div>
              </div>
            ))}
          </div>
        )}
      </div>
    );
  };

  const SettingsView = () => (
    <div className="animate-in fade-in slide-in-from-right-4 duration-300 pb-24 h-full space-y-6">
      <h2 className="text-xl font-bold text-slate-800 dark:text-slate-100 flex items-center gap-2">
        <Settings className="text-indigo-500" /> Ajustes e Segurança
      </h2>

      <div className="bg-white dark:bg-slate-800 p-5 rounded-3xl shadow-sm border border-slate-100 dark:border-slate-700 space-y-4">
        <div className="flex items-center gap-3">
          <div className="w-12 h-12 rounded-full bg-indigo-100 dark:bg-indigo-900/40 text-indigo-600 dark:text-indigo-400 flex items-center justify-center font-bold text-lg">
            {userName.charAt(0)}
          </div>
          <div>
            <h3 className="font-bold text-slate-800 dark:text-slate-100">Perfil de Utilizador</h3>
            <p className="text-xs text-slate-500 dark:text-slate-400">Nome exibido no topo</p>
          </div>
        </div>
        <form onSubmit={(e) => { e.preventDefault(); setUserName(tempUserName); alert("Nome guardado!"); }} className="space-y-3 pt-2">
          <div className="flex items-center bg-slate-50 dark:bg-slate-700/50 rounded-xl px-3 py-2 border border-slate-200 dark:border-slate-600">
            <User size={18} className="text-slate-400 mr-2" />
            <input type="text" value={tempUserName} onChange={(e) => setTempUserName(e.target.value)} className="bg-transparent flex-1 outline-none text-slate-800 dark:text-slate-200 text-sm" />
          </div>
          <button type="submit" className="w-full py-2.5 bg-indigo-600 text-white font-bold rounded-xl text-sm transition-colors">Guardar Nome</button>
        </form>
      </div>

      <div className="bg-white dark:bg-slate-800 p-5 rounded-3xl shadow-sm border border-slate-100 dark:border-slate-700 space-y-3">
        <h3 className="font-bold text-slate-800 dark:text-slate-100 text-sm flex items-center gap-2"><DollarSign size={16}/> Chave PIX Padrão para Cobrança</h3>
        <input type="text" value={pixKey} onChange={(e) => setPixKey(e.target.value)} className="w-full bg-slate-50 dark:bg-slate-700/50 rounded-xl px-3 py-2 border border-slate-200 dark:border-slate-600 text-sm outline-none text-slate-800 dark:text-slate-200" />
      </div>

      <div className="bg-white dark:bg-slate-800 p-5 rounded-3xl shadow-sm border border-slate-100 dark:border-slate-700 space-y-4">
        <div className="flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="p-2.5 bg-amber-100 dark:bg-amber-500/20 text-amber-600 rounded-2xl"><Percent size={20} /></div>
            <div>
              <h3 className="font-bold text-slate-800 dark:text-slate-100 text-sm">Juros de Mora Diários</h3>
              <p className="text-xs text-slate-500 dark:text-slate-400">Correção em atrasos</p>
            </div>
          </div>
          <input type="checkbox" checked={useInterest} onChange={(e) => setUseInterest(e.target.checked)} className="w-5 h-5 accent-indigo-600 cursor-pointer" />
        </div>
        {useInterest && (
          <div className="pt-2">
            <label className="text-xs font-semibold text-slate-500">Taxa ao Mês (%)</label>
            <input type="number" step="0.1" value={interestRate} onChange={(e) => setInterestRate(e.target.value)} className="w-full bg-slate-50 dark:bg-slate-700/50 rounded-xl px-3 py-2 border border-slate-600 mt-1 text-sm text-slate-800 dark:text-slate-200 outline-none" />
          </div>
        )}
      </div>

      <div className="bg-white dark:bg-slate-800 p-5 rounded-3xl shadow-sm border border-slate-100 dark:border-slate-700 space-y-4">
        <div className="flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="p-2.5 bg-indigo-100 dark:bg-indigo-500/20 text-indigo-600 rounded-2xl"><Shield size={20} /></div>
            <div>
              <h3 className="font-bold text-slate-800 dark:text-slate-100 text-sm">Bloqueio por PIN</h3>
              <p className="text-xs text-slate-500 dark:text-slate-400">PIN padrão: 1234 | Cofre: 0000</p>
            </div>
          </div>
          <input type="checkbox" checked={usePin} onChange={(e) => { setUsePin(e.target.checked); if(e.target.checked) setIsLocked(true); }} className="w-5 h-5 accent-indigo-600 cursor-pointer" />
        </div>
      </div>

      <div className="bg-white dark:bg-slate-800 p-5 rounded-3xl shadow-sm border border-slate-100 dark:border-slate-700 space-y-3">
        <h3 className="font-bold text-slate-800 dark:text-slate-100 text-sm flex items-center gap-2"><Download size={16} /> Exportar Relatório</h3>
        <button onClick={exportData} className="w-full py-3 bg-slate-100 dark:bg-slate-700 text-slate-800 dark:text-white font-bold rounded-xl text-sm transition-colors">Descarregar JSON</button>
      </div>
    </div>
  );

  if (usePin && isLocked) {
    return (
      <div className={`min-h-screen bg-slate-900 flex items-center justify-center p-4 font-sans text-white`}>
        <div className="w-full max-w-sm bg-slate-800 rounded-3xl p-8 shadow-2xl text-center space-y-6">
          <div className="w-16 h-16 bg-indigo-600/20 text-indigo-400 rounded-full flex items-center justify-center mx-auto">
            <Lock size={32} />
          </div>
          <div>
            <h2 className="text-xl font-bold mb-1">Aplicativo Bloqueado</h2>
            <p className="text-xs text-slate-400">Insira o seu PIN de segurança para continuar</p>
          </div>
          <form onSubmit={handlePinSubmit} className="space-y-4">
            <input 
              type="password" maxLength="4" value={enteredPin} onChange={(e) => setEnteredPin(e.target.value)} 
              placeholder="••••" className="w-full bg-slate-900 text-center text-2xl tracking-widest py-3 rounded-xl border border-slate-700 outline-none"
            />
            <button type="submit" className="w-full py-3 bg-indigo-600 font-bold rounded-xl shadow-lg hover:bg-indigo-700 transition-colors">Desbloquear</button>
          </form>
        </div>
      </div>
    );
  }

  return (
    <div className={`min-h-screen bg-slate-100 dark:bg-black p-0 sm:p-4 flex items-center justify-center font-sans ${darkMode ? 'dark' : ''}`}>
      <div className="w-full h-[100dvh] sm:h-[850px] max-w-[400px] bg-slate-50 dark:bg-slate-900 sm:rounded-[40px] sm:shadow-2xl relative overflow-hidden flex flex-col border-[8px] border-slate-800 dark:border-black">
        
        <div className="px-6 pt-12 pb-4 flex items-center justify-between bg-slate-50/90 dark:bg-slate-900/90 backdrop-blur-md z-10 sticky top-0">
          <div className="flex items-center gap-3">
            <div className="w-10 h-10 rounded-full bg-indigo-600 flex items-center justify-center text-white font-bold shadow-lg">
              {userName.charAt(0)}
            </div>
            <div>
              <p className="text-xs text-slate-500 dark:text-slate-400">Olá,</p>
              <h1 className="text-lg font-bold text-slate-800 dark:text-slate-100 leading-tight truncate max-w-[140px]">{userName}</h1>
            </div>
          </div>
          <div className="flex gap-2">
            {usePin && (
              <button onClick={() => setIsLocked(true)} className="p-2 bg-white dark:bg-slate-800 rounded-full shadow-sm border border-slate-100 dark:border-slate-700 text-indigo-500">
                <Lock size={18} />
              </button>
            )}
            <button onClick={() => setDarkMode(!darkMode)} className="p-2 bg-white dark:bg-slate-800 rounded-full shadow-sm border border-slate-100 dark:border-slate-700 text-slate-600 dark:text-slate-300">
              {darkMode ? <Sun size={18} /> : <Moon size={18} />}
            </button>
          </div>
        </div>

        <div className="flex-1 overflow-y-auto px-6 pt-2 hide-scrollbar relative">
          {activeTab === 'home' && <DashboardView />}
          {activeTab === 'calendar' && <CalendarView />}
          {activeTab === 'ranking' && <RankingView />}
          {activeTab === 'settings' && <SettingsView />}
        </div>

        <div className="absolute bottom-0 left-0 right-0 h-20 bg-white dark:bg-slate-900 border-t border-slate-200 dark:border-slate-800 px-6 flex items-center justify-between pb-4">
          <NavItem icon={<Home />} label="Início" isActive={activeTab === 'home'} onClick={() => setActiveTab('home')} />
          <NavItem icon={<CalendarIcon />} label="Agenda" isActive={activeTab === 'calendar'} onClick={() => setActiveTab('calendar')} />
          <div className="relative -top-5">
            <button onClick={() => { setIsAddModalOpen(true); setIsEditing(false); setSelectedDebt(null); }} className="w-14 h-14 bg-indigo-600 rounded-full flex items-center justify-center text-white shadow-xl shadow-indigo-600/40 hover:scale-105 active:scale-95 transition-transform"><Plus size={28} /></button>
          </div>
          <NavItem icon={<Trophy />} label="Ranking" isActive={activeTab === 'ranking'} onClick={() => setActiveTab('ranking')} />
          <NavItem icon={<Settings />} label="Ajustes" isActive={activeTab === 'settings'} onClick={() => setActiveTab('settings')} />
        </div>

        {(isAddModalOpen || isEditing) && (
          <div className="absolute inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-end animate-in fade-in duration-200">
            <div className="w-full bg-white dark:bg-slate-900 rounded-t-[32px] p-6 pb-12 animate-in slide-in-from-bottom-full duration-300">
              <div className="flex justify-between items-center mb-6">
                <h2 className="text-xl font-bold text-slate-800 dark:text-white">{isEditing ? 'Editar Dívida' : 'Nova Dívida'}</h2>
                <button onClick={() => { setIsAddModalOpen(false); setIsEditing(false); }} className="p-2 bg-slate-100 dark:bg-slate-800 rounded-full text-slate-500"><X size={20} /></button>
              </div>

              <form onSubmit={handleSaveDebt} className="space-y-4">
                <div>
                  <label className="text-xs font-semibold text-slate-500 ml-1">Devedor</label>
                  <div className="flex items-center bg-slate-50 dark:bg-slate-800 rounded-xl p-3 border border-slate-200 dark:border-slate-700 mt-1">
                    <User size={18} className="text-slate-400 mr-2" />
                    <input required name="name" defaultValue={isEditing ? selectedDebt?.name : ''} type="text" placeholder="Nome" className="bg-transparent flex-1 outline-none text-slate-800 dark:text-slate-200" />
                  </div>
                </div>
                <div className="grid grid-cols-2 gap-4">
                  <div>
                    <label className="text-xs font-semibold text-slate-500 ml-1">Valor (R$)</label>
                    <div className="flex items-center bg-slate-50 dark:bg-slate-800 rounded-xl p-3 border border-slate-200 dark:border-slate-700 mt-1">
                      <CreditCard size={18} className="text-slate-400 mr-2" />
                      <input required name="amount" defaultValue={isEditing ? selectedDebt?.amount : ''} type="number" step="0.01" min="1" placeholder="0.00" className="bg-transparent flex-1 outline-none text-slate-800 dark:text-slate-200 w-full" />
                    </div>
                  </div>
                  <div>
                    <label className="text-xs font-semibold text-slate-500 ml-1">Vencimento</label>
                    <div className="flex items-center bg-slate-50 dark:bg-slate-800 rounded-xl p-3 border border-slate-200 dark:border-slate-700 mt-1">
                      <input required name="dueDate" type="date" defaultValue={isEditing ? selectedDebt?.dueDate : new Date().toISOString().split('T')[0]} className="bg-transparent flex-1 outline-none text-slate-800 dark:text-slate-200 w-full text-sm" />
                    </div>
                  </div>
                </div>
                <div>
                  <label className="text-xs font-semibold text-slate-500 ml-1">Motivo</label>
                  <div className="flex items-center bg-slate-50 dark:bg-slate-800 rounded-xl p-3 border border-slate-200 dark:border-slate-700 mt-1">
                    <FileText size={18} className="text-slate-400 mr-2" />
                    <input required name="type" defaultValue={isEditing ? selectedDebt?.type : ''} type="text" placeholder="Ex: Empréstimo" className="bg-transparent flex-1 outline-none text-slate-800 dark:text-slate-200" />
                  </div>
                </div>
                <div>
                  <label className="text-xs font-semibold text-slate-500 ml-1">WhatsApp</label>
                  <div className="flex items-center bg-slate-50 dark:bg-slate-800 rounded-xl p-3 border border-slate-200 dark:border-slate-700 mt-1">
                    <Phone size={18} className="text-slate-400 mr-2" />
                    <input name="phone" defaultValue={isEditing ? selectedDebt?.phone : ''} type="tel" placeholder="5511912345678" className="bg-transparent flex-1 outline-none text-slate-800 dark:text-slate-200" />
                  </div>
                </div>
                <button type="submit" className="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-4 rounded-xl mt-4 transition-colors">Guardar Dívida</button>
              </form>
            </div>
          </div>
        )}

        {selectedDebt && !isEditing && (
          <div className="absolute inset-0 bg-slate-900/60 backdrop-blur-sm z-40 flex flex-col justify-end animate-in fade-in duration-200">
            <div className="w-full h-[90%] bg-slate-50 dark:bg-slate-900 rounded-t-[32px] flex flex-col shadow-2xl animate-in slide-in-from-bottom-full duration-300">
              
              <div className="px-6 py-4 flex justify-between items-center bg-white dark:bg-slate-800 rounded-t-[32px] border-b border-slate-100 dark:border-slate-700">
                <button onClick={() => setSelectedDebt(null)} className="p-2 bg-slate-100 dark:bg-slate-700 rounded-full text-slate-500"><ChevronLeft size={20} /></button>
                <h3 className="font-bold text-slate-800 dark:text-white">Perfil e Amortização</h3>
                <div className="flex gap-2">
                  <button onClick={() => setDebtToDelete(selectedDebt.id)} className="p-2 text-rose-500 bg-rose-50 dark:bg-rose-500/10 rounded-full"><Trash2 size={18}/></button>
                  <button onClick={() => setIsEditing(true)} className="p-2 text-indigo-600 bg-indigo-50 dark:bg-indigo-500/10 rounded-full"><Edit3 size={18}/></button>
                </div>
              </div>

              <div className="flex-1 overflow-y-auto px-6 py-6 space-y-6 hide-scrollbar">
                <div className="text-center">
                  <div className="w-16 h-16 mx-auto rounded-full bg-indigo-100 dark:bg-indigo-900/40 text-indigo-600 dark:text-indigo-400 flex items-center justify-center text-2xl font-bold mb-3">
                    {selectedDebt.name.charAt(0)}
                  </div>
                  <h2 className="text-2xl font-bold text-slate-800 dark:text-slate-100">{selectedDebt.name}</h2>
                  <p className="text-slate-500 dark:text-slate-400">{selectedDebt.type}</p>
                </div>

                <div className="bg-white dark:bg-slate-800 p-5 rounded-2xl border border-slate-100 dark:border-slate-700 shadow-sm flex justify-between items-center">
                  <div>
                    <p className="text-xs text-slate-500 mb-1">Restante a Pagar</p>
                    <h3 className="text-2xl font-bold text-indigo-600 dark:text-indigo-400">{formatCurrency(selectedDebt.calculatedRemaining)}</h3>
                    <p className="text-[10px] text-slate-400">Original: {formatCurrency(selectedDebt.amount)}</p>
                  </div>
                  <div className="text-right">
                    <p className="text-xs text-slate-500 mb-1">Vencimento</p>
                    <h3 className="text-sm font-bold text-slate-800 dark:text-slate-200">{formatDateBr(selectedDebt.dueDate)}</h3>
                  </div>
                </div>

                {!selectedDebt.isPaid && (
                  <div className="bg-indigo-50/50 dark:bg-indigo-500/10 p-4 rounded-2xl border border-indigo-100 dark:border-indigo-500/20 space-y-3">
                    <h4 className="font-bold text-xs text-indigo-600 dark:text-indigo-400 uppercase">Registar Pagamento Parcial (Amortização)</h4>
                    <form onSubmit={handlePartialPayment} className="flex gap-2">
                      <input 
                        type="number" step="0.01" min="1" max={selectedDebt.remaining} value={partialAmount} onChange={(e) => setPartialAmount(e.target.value)}
                        placeholder="Valor pago (R$)" className="flex-1 bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl px-3 py-2 text-sm outline-none text-slate-800 dark:text-slate-200"
                      />
                      <button type="submit" className="bg-indigo-600 text-white px-4 rounded-xl font-bold text-xs hover:bg-indigo-700">Abater</button>
                    </form>
                  </div>
                )}

                <div className="bg-white dark:bg-slate-800 p-4 rounded-2xl border border-slate-100 dark:border-slate-700 space-y-3">
                  <h4 className="font-bold text-xs text-slate-500 uppercase flex items-center gap-1"><Sparkles size={14} className="text-amber-500"/> Estilo de Cobrança WhatsApp</h4>
                  <div className="grid grid-cols-3 gap-2">
                    <button onClick={() => setSelectedTone('friendly')} className={`py-1.5 rounded-xl text-xs font-semibold ${selectedTone === 'friendly' ? 'bg-indigo-600 text-white' : 'bg-slate-100 dark:bg-slate-700 text-slate-600 dark:text-slate-300'}`}>Amigável</button>
                    <button onClick={() => setSelectedTone('formal')} className={`py-1.5 rounded-xl text-xs font-semibold ${selectedTone === 'formal' ? 'bg-indigo-600 text-white' : 'bg-slate-100 dark:bg-slate-700 text-slate-600 dark:text-slate-300'}`}>Formal</button>
                    <button onClick={() => setSelectedTone('funny')} className={`py-1.5 rounded-xl text-xs font-semibold ${selectedTone === 'funny' ? 'bg-indigo-600 text-white' : 'bg-slate-100 dark:bg-slate-700 text-slate-600 dark:text-slate-300'}`}>Humor</button>
                  </div>
                  <button onClick={(e) => handleWhatsApp(selectedDebt, e)} className="w-full mt-2 bg-emerald-600 hover:bg-emerald-700 text-white py-2.5 rounded-xl text-sm font-bold flex items-center justify-center gap-2">
                    <MessageCircle size={16} /> Enviar Mensagem Pronta
                  </button>
                </div>

                <div className="bg-white dark:bg-slate-800 p-4 rounded-2xl border border-slate-100 dark:border-slate-700 space-y-3">
                  <div className="flex justify-between items-center">
                    <h4 className="font-bold text-xs text-slate-500 uppercase flex items-center gap-1"><Paperclip size={14} /> Provas e Comprovantes</h4>
                    <button onClick={handleFileUploadSim} className="text-xs text-indigo-600 dark:text-indigo-400 font-bold hover:underline">+ Adicionar Print</button>
                  </div>
                  <div className="flex gap-2 overflow-x-auto pb-1">
                    {selectedDebt.attachments && selectedDebt.attachments.length > 0 ? (
                      selectedDebt.attachments.map((img, idx) => (
                        <div key={idx} className="w-20 h-20 rounded-xl overflow-hidden border border-slate-200 dark:border-slate-700 shrink-0 bg-slate-100">
                          <img src={img} alt="Comprovante" className="w-full h-full object-cover" />
                        </div>
                      ))
                    ) : (
                      <p className="text-xs text-slate-400 italic">Nenhum print ou recibo anexado.</p>
                    )}
                  </div>
                </div>

                <div>
                  <h4 className="font-bold text-slate-800 dark:text-slate-200 flex items-center gap-2 mb-4">
                    <Clock size={18} className="text-indigo-500" /> Diário de Interações
                  </h4>
                  <div className="space-y-3 mb-4">
                    {selectedDebt.logs && selectedDebt.logs.length > 0 ? (
                      selectedDebt.logs.map((log, index) => (
                        <div key={index} className="bg-white dark:bg-slate-800 p-3 rounded-xl border border-slate-100 dark:border-slate-700 relative ml-4">
                          <div className="absolute -left-5 top-4 w-2 h-2 rounded-full bg-indigo-400"></div>
                          <p className="text-xs text-slate-400 mb-1">{formatDateBr(log.date)}</p>
                          <p className="text-sm text-slate-700 dark:text-slate-300">{log.text}</p>
                        </div>
                      ))
                    ) : (
                      <p className="text-sm text-slate-500 italic text-center py-4 bg-slate-100 dark:bg-slate-800/50 rounded-xl">Sem interações anotadas.</p>
                    )}
                  </div>
                  <form onSubmit={handleAddLog} className="flex gap-2">
                    <input 
                      type="text" value={newLogText} onChange={(e) => setNewLogText(e.target.value)}
                      placeholder="O que ele(a) disse hoje?" className="flex-1 bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl px-4 py-3 text-sm outline-none text-slate-800 dark:text-slate-200"
                    />
                    <button type="submit" className="bg-indigo-600 text-white rounded-xl px-4 flex items-center justify-center hover:bg-indigo-700"><Send size={18} /></button>
                  </form>
                </div>
              </div>
            </div>
          </div>
        )}

        {debtToDelete && (
          <div className="absolute inset-0 bg-slate-900/60 backdrop-blur-sm z-[60] flex items-center justify-center p-6 animate-in fade-in duration-200">
            <div className="bg-white dark:bg-slate-800 rounded-3xl p-6 w-full max-w-sm shadow-2xl text-center">
              <div className="w-16 h-16 bg-rose-100 dark:bg-rose-500/20 text-rose-600 rounded-full flex items-center justify-center mx-auto mb-4">
                <AlertCircle size={32} />
              </div>
              <h3 className="text-xl font-bold text-slate-800 dark:text-slate-100 mb-2">Eliminar Dívida?</h3>
              <p className="text-sm text-slate-500 dark:text-slate-400 mb-6">Tem a certeza de que deseja apagar permanentemente?</p>
              <div className="flex gap-3">
                <button onClick={() => setDebtToDelete(null)} className="flex-1 py-3 bg-slate-100 dark:bg-slate-700 text-slate-700 dark:text-slate-300 rounded-xl font-bold">Cancelar</button>
                <button onClick={confirmDelete} className="flex-1 py-3 bg-rose-600 text-white rounded-xl font-bold hover:bg-rose-700">Sim, Eliminar</button>
              </div>
            </div>
          </div>
        )}

      </div>
      <style>{`.hide-scrollbar::-webkit-scrollbar { display: none; } .hide-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }`}</style>
    </div>
  );
}
