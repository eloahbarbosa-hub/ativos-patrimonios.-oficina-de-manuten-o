import React, { useState, useEffect, useRef } from 'react';
import { 
  Mail, Lock, UserPlus, LogIn, LogOut, 
  AlertCircle, CheckCircle2, Home, Package, 
  Wrench, Box, ChevronRight, MapPin, ArrowLeft,
  Plus, Trash2, X, QrCode, Camera, FileText, Link as LinkIcon
} from 'lucide-react';

export default function App() {
  // --- ESTADOS DE AUTENTICAÇÃO ---
  const [users, setUsers] = useState(() => {
    const saved = localStorage.getItem('senaiUsers');
    return saved ? JSON.parse(saved) : [];
  });
  const [currentView, setCurrentView] = useState('login'); 
  const [currentUser, setCurrentUser] = useState(() => {
    const saved = localStorage.getItem('senaiLoggedUser');
    return saved ? JSON.parse(saved) : null;
  });

  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');
  const [success, setSuccess] = useState('');

  // --- ESTADOS DO APLICATIVO SENAI ---
  const [inventory, setInventory] = useState(() => {
    const saved = localStorage.getItem('senaiInventory');
    return saved ? JSON.parse(saved) : [
      { id: 1, name: 'Filtro de Óleo EA111', qty: 15, nis: 'NIS-847291' },
      { id: 2, name: 'Pastilha de Freio Bosch', qty: 8, nis: 'NIS-392810' }
    ];
  });

  const [tools, setTools] = useState(() => {
    const saved = localStorage.getItem('senaiTools');
    return saved ? JSON.parse(saved) : [
      { id: 1, name: 'Torquímetro Gedore', loc: 'Gaveta A1', nis: 'NIS-102938' },
      { id: 2, name: 'Scanner OBD2', loc: 'Bancada 04', nis: 'NIS-564738' }
    ];
  });

  const [manuals, setManuals] = useState(() => {
    const saved = localStorage.getItem('senaiManuals');
    return saved ? JSON.parse(saved) : [
      { id: 1, name: 'Esquema Elétrico Gol G6', url: '#' },
      { id: 2, name: 'Tabela de Torques Motores VW', url: '#' }
    ];
  });

  // --- ESTADOS DE FORMULÁRIO (Adicionar Itens) ---
  const [showAddInventory, setShowAddInventory] = useState(false);
  const [newItemName, setNewItemName] = useState('');
  const [newItemQty, setNewItemQty] = useState('');
  const [newItemNis, setNewItemNis] = useState('');

  const [showAddTool, setShowAddTool] = useState(false);
  const [newToolName, setNewToolName] = useState('');
  const [newToolLoc, setNewToolLoc] = useState('');
  const [newToolNis, setNewToolNis] = useState('');

  const [showAddManual, setShowAddManual] = useState(false);
  const [newManualName, setNewManualName] = useState('');
  const [newManualUrl, setNewManualUrl] = useState('');

  // --- ESTADOS DO SCANNER QR CODE ---
  const [isScanning, setIsScanning] = useState(null); 
  const videoRef = useRef(null);
  const [stream, setStream] = useState(null);

  // Salva dados no LocalStorage
  useEffect(() => {
    localStorage.setItem('senaiUsers', JSON.stringify(users));
    localStorage.setItem('senaiInventory', JSON.stringify(inventory));
    localStorage.setItem('senaiTools', JSON.stringify(tools));
    localStorage.setItem('senaiManuals', JSON.stringify(manuals));
    if (currentUser) {
      localStorage.setItem('senaiLoggedUser', JSON.stringify(currentUser));
    } else {
      localStorage.removeItem('senaiLoggedUser');
    }
  }, [users, inventory, tools, manuals, currentUser]);

  useEffect(() => {
    if (currentUser && (currentView === 'login' || currentView === 'register')) {
      setCurrentView('home');
    }
  }, [currentUser]);

  useEffect(() => {
    return () => stopScanner();
  }, [currentView]);

  // --- FUNÇÕES DE AUTENTICAÇÃO ---
  const handleRegister = (e) => {
    e.preventDefault();
    if (!email || !password) { setError('Preencha todos os campos.'); return; }
    if (users.some(u => u.email === email)) { setError('Email já cadastrado!'); return; }
    setUsers([...users, { email, password }]);
    setPassword('');
    setSuccess('Conta criada com sucesso!');
    setCurrentView('login');
  };

  const handleLogin = (e) => {
    e.preventDefault();
    const user = users.find(u => u.email === email && u.password === password);
    if (user) { setCurrentUser(user); setCurrentView('home'); } 
    else { setError('Email ou senha incorretos.'); }
  };

  const handleLogout = () => {
    setCurrentUser(null);
    setCurrentView('login');
  };

  // --- FUNÇÕES DE GERENCIAMENTO ---
  const addInventoryItem = (e) => {
    e.preventDefault();
    if(!newItemName || !newItemQty) return;
    setInventory([...inventory, { id: Date.now(), name: newItemName, qty: parseInt(newItemQty), nis: newItemNis || 'Sem NIS' }]);
    setNewItemName(''); setNewItemQty(''); setNewItemNis(''); setShowAddInventory(false);
  };

  const addToolItem = (e) => {
    e.preventDefault();
    if(!newToolName || !newToolLoc) return;
    setTools([...tools, { id: Date.now(), name: newToolName, loc: newToolLoc, nis: newToolNis || 'Sem NIS' }]);
    setNewToolName(''); setNewToolLoc(''); setNewToolNis(''); setShowAddTool(false);
  };

  const addManualItem = (e) => {
    e.preventDefault();
    if(!newManualName) return;
    setManuals([...manuals, { id: Date.now(), name: newManualName, url: newManualUrl || '#' }]);
    setNewManualName(''); setNewManualUrl(''); setShowAddManual(false);
  };

  const deleteItem = (id, type) => {
    if (type === 'inventory') setInventory(inventory.filter(i => i.id !== id));
    if (type === 'tools') setTools(tools.filter(i => i.id !== id));
    if (type === 'manuals') setManuals(manuals.filter(i => i.id !== id));
  };

  // --- FUNÇÕES DO SCANNER ---
  const startScanner = async (type) => {
    setIsScanning(type);
    try {
      const mediaStream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
      setStream(mediaStream);
      setTimeout(() => { if (videoRef.current) videoRef.current.srcObject = mediaStream; }, 100);
    } catch (err) {
      alert("Erro ao aceder à câmara.");
      setIsScanning(null);
    }
  };

  const stopScanner = () => {
    if (stream) stream.getTracks().forEach(track => track.stop());
    setStream(null); setIsScanning(null);
  };

  const handleSimulateScan = () => {
    const mockNIS = `NIS-${Math.floor(100000 + Math.random() * 900000)}`;
    if (isScanning === 'inventory') setNewItemNis(mockNIS);
    else if (isScanning === 'tool') setNewToolNis(mockNIS);
    else if (isScanning === 'search') alert(`Lido: ${mockNIS}`);
    stopScanner();
  };

  // --- COMPONENTES DE INTERFACE ---
  const renderScannerModal = () => {
    if (!isScanning) return null;
    return (
      <div className="fixed inset-0 z-[100] bg-black flex flex-col items-center justify-center">
        <div className="absolute top-0 w-full p-6 flex justify-between items-center bg-gradient-to-b from-black/70 to-transparent">
          <h3 className="text-white font-bold text-lg">Escanear NIS</h3>
          <button onClick={stopScanner} className="text-white p-2 bg-white/20 rounded-full"><X size={24} /></button>
        </div>
        <div className="relative w-full h-full flex items-center justify-center overflow-hidden">
          <video ref={videoRef} autoPlay playsInline className="absolute inset-0 w-full h-full object-cover" />
          <div className="relative z-10 w-64 h-64 border-2 border-white/50 rounded-xl">
             <div className="absolute top-0 left-0 w-8 h-8 border-t-4 border-l-4 border-red-500 rounded-tl-xl"></div>
             <div className="w-full h-0.5 bg-red-500/50 animate-pulse absolute top-1/2"></div>
          </div>
        </div>
        <div className="absolute bottom-10 w-full px-6">
          <button onClick={handleSimulateScan} className="w-full bg-red-600 text-white font-bold py-4 rounded-xl flex items-center justify-center gap-2">
            <QrCode size={20} /> Simular Leitura
          </button>
        </div>
      </div>
    );
  };

  if (currentView === 'login' || currentView === 'register') {
    return (
      <div className="min-h-screen bg-slate-100 flex items-center justify-center p-4">
        <div className="bg-white rounded-2xl shadow-xl w-full max-w-md p-6">
          <div className="text-center mb-8">
            <h1 className="text-2xl font-bold text-red-600">SENAI Oficina Digital</h1>
            <p className="text-slate-500 text-sm">Plataforma de Gestão Automotiva</p>
          </div>
          <form onSubmit={currentView === 'login' ? handleLogin : handleRegister} className="space-y-4">
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" className="w-full p-3 border rounded-lg outline-none focus:ring-2 focus:ring-red-500" />
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Senha" className="w-full p-3 border rounded-lg outline-none focus:ring-2 focus:ring-red-500" />
            <button type="submit" className="w-full bg-red-600 text-white p-3 rounded-lg font-bold">{currentView === 'login' ? 'Entrar' : 'Registrar'}</button>
          </form>
          <button onClick={() => setCurrentView(currentView === 'login' ? 'register' : 'login')} className="w-full text-center mt-4 text-sm text-red-600 font-medium">
            {currentView === 'login' ? 'Criar uma conta' : 'Já tenho conta'}
          </button>
        </div>
      </div>
    );
  }

  return (
    <div className="bg-slate-100 min-h-screen font-sans">
      {renderScannerModal()}
      <div className="max-w-md mx-auto min-h-screen bg-gray-50 relative pb-24 shadow-2xl">
        <header className="bg-white p-6 shadow-sm rounded-b-3xl flex justify-between items-center mb-6">
          <div className="truncate pr-4">
            <h2 className="text-gray-500 text-sm font-medium">Bem-vindo(a),</h2>
            <h3 className="text-lg font-bold text-gray-900 truncate">{currentUser?.email}</h3>
          </div>
          <button onClick={handleLogout} className="p-3 bg-red-50 text-red-600 rounded-xl shrink-0"><LogOut size={20} /></button>
        </header>

        <main className="px-6">
          {currentView === 'home' && (
            <div className="space-y-6 animate-in fade-in">
              <div className="grid grid-cols-2 gap-4">
                <div className="bg-red-600 p-5 rounded-3xl text-white shadow-lg shadow-red-600/20">
                  <Package className="mb-2 opacity-80" size={20} />
                  <p className="text-xs opacity-90">Peças</p>
                  <h4 className="text-2xl font-bold">{inventory.length}</h4>
                </div>
                <div className="bg-slate-800 p-5 rounded-3xl text-white shadow-lg shadow-slate-800/20">
                  <Wrench className="mb-2 opacity-80" size={20} />
                  <p className="text-xs opacity-90">Recursos</p>
                  <h4 className="text-2xl font-bold">{tools.length}</h4>
                </div>
              </div>

              <button onClick={() => startScanner('search')} className="w-full bg-slate-900 text-white p-4 rounded-2xl flex items-center justify-center gap-3 active:scale-95 transition-all">
                <Camera size={24} className="text-red-500" />
                <span className="font-bold">Escanear QR Code</span>
              </button>

              <div className="space-y-3">
                <h4 className="font-bold text-slate-800">Acesso Rápido</h4>
                <div className="grid grid-cols-1 gap-3">
                  <button onClick={() => setCurrentView('inventory')} className="bg-white p-4 rounded-2xl flex items-center gap-4 shadow-sm border border-slate-100">
                    <div className="w-10 h-10 bg-red-50 text-red-600 rounded-xl flex items-center justify-center"><Box size={20} /></div>
                    <span className="font-bold text-slate-700">Estoque</span>
                    <ChevronRight className="ml-auto text-slate-300" size={20} />
                  </button>
                  <button onClick={() => setCurrentView('tools')} className="bg-white p-4 rounded-2xl flex items-center gap-4 shadow-sm border border-slate-100">
                    <div className="w-10 h-10 bg-slate-100 text-slate-600 rounded-xl flex items-center justify-center"><MapPin size={20} /></div>
                    <span className="font-bold text-slate-700">Ferramentas</span>
                    <ChevronRight className="ml-auto text-slate-300" size={20} />
                  </button>
                  <button onClick={() => setCurrentView('manuals')} className="bg-white p-4 rounded-2xl flex items-center gap-4 shadow-sm border border-slate-100">
                    <div className="w-10 h-10 bg-blue-50 text-blue-600 rounded-xl flex items-center justify-center"><FileText size={20} /></div>
                    <span className="font-bold text-slate-700">Manuais Técnicos</span>
                    <ChevronRight className="ml-auto text-slate-300" size={20} />
                  </button>
                </div>
              </div>
            </div>
          )}

          {currentView === 'inventory' && (
            <div className="animate-in fade-in">
              <div className="flex items-center justify-between mb-6">
                <div className="flex items-center gap-3">
                  <button onClick={() => setCurrentView('home')} className="p-2 bg-white rounded-full"><ArrowLeft size={20} /></button>
                  <h2 className="text-2xl font-bold text-slate-800">Estoque</h2>
                </div>
                <button onClick={() => setShowAddInventory(!showAddInventory)} className="p-2 bg-red-600 text-white rounded-xl">
                  {showAddInventory ? <X size={20} /> : <Plus size={20} />}
                </button>
              </div>
              {showAddInventory && (
                <form onSubmit={addInventoryItem} className="bg-white p-4 rounded-2xl shadow-sm space-y-3 mb-6">
                  <input type="text" placeholder="Nome da peça" value={newItemName} onChange={(e) => setNewItemName(e.target.value)} className="w-full p-2 border rounded" />
                  <div className="flex gap-2">
                    <input type="text" placeholder="NIS" value={newItemNis} onChange={(e) => setNewItemNis(e.target.value)} className="flex-1 p-2 border rounded" />
                    <button type="button" onClick={() => startScanner('inventory')} className="p-2 bg-slate-100 rounded text-slate-500"><QrCode size={18} /></button>
                  </div>
                  <input type="number" placeholder="Quantidade" value={newItemQty} onChange={(e) => setNewItemQty(e.target.value)} className="w-full p-2 border rounded" />
                  <button type="submit" className="w-full bg-slate-800 text-white p-2 rounded font-bold">Salvar</button>
                </form>
              )}
              <div className="space-y-3">
                {inventory.map(item => (
                  <div key={item.id} className="bg-white p-4 rounded-2xl flex justify-between items-center shadow-sm">
                    <div>
                      <p className="font-bold text-slate-800">{item.name}</p>
                      <p className="text-xs text-slate-500">NIS: {item.nis} • Qtd: {item.qty}</p>
                    </div>
                    <button onClick={() => deleteItem(item.id, 'inventory')} className="text-slate-300 hover:text-red-600"><Trash2 size={18} /></button>
                  </div>
                ))}
              </div>
            </div>
          )}

          {currentView === 'tools' && (
            <div className="animate-in fade-in">
              <div className="flex items-center justify-between mb-6">
                <div className="flex items-center gap-3">
                  <button onClick={() => setCurrentView('home')} className="p-2 bg-white rounded-full"><ArrowLeft size={20} /></button>
                  <h2 className="text-2xl font-bold text-slate-800">Ferramentas</h2>
                </div>
                <button onClick={() => setShowAddTool(!showAddTool)} className="p-2 bg-red-600 text-white rounded-xl">
                  {showAddTool ? <X size={20} /> : <Plus size={20} />}
                </button>
              </div>
              {showAddTool && (
                <form onSubmit={addToolItem} className="bg-white p-4 rounded-2xl shadow-sm space-y-3 mb-6">
                  <input type="text" placeholder="Nome" value={newToolName} onChange={(e) => setNewToolName(e.target.value)} className="w-full p-2 border rounded" />
                  <input type="text" placeholder="Localização" value={newToolLoc} onChange={(e) => setNewToolLoc(e.target.value)} className="w-full p-2 border rounded" />
                  <button type="submit" className="w-full bg-slate-800 text-white p-2 rounded font-bold">Salvar</button>
                </form>
              )}
              <div className="space-y-3">
                {tools.map(item => (
                  <div key={item.id} className="bg-white p-4 rounded-2xl flex justify-between items-center shadow-sm">
                    <div>
                      <p className="font-bold text-slate-800">{item.name}</p>
                      <p className="text-xs text-slate-500">{item.loc}</p>
                    </div>
                    <button onClick={() => deleteItem(item.id, 'tools')} className="text-slate-300 hover:text-red-600"><Trash2 size={18} /></button>
                  </div>
                ))}
              </div>
            </div>
          )}

          {currentView === 'manuals' && (
            <div className="animate-in fade-in">
              <div className="flex items-center justify-between mb-6">
                <div className="flex items-center gap-3">
                  <button onClick={() => setCurrentView('home')} className="p-2 bg-white rounded-full"><ArrowLeft size={20} /></button>
                  <h2 className="text-2xl font-bold text-slate-800">Manuais</h2>
                </div>
                <button onClick={() => setShowAddManual(!showAddManual)} className="p-2 bg-blue-600 text-white rounded-xl shadow-lg">
                  {showAddManual ? <X size={20} /> : <Plus size={20} />}
                </button>
              </div>

              {showAddManual && (
                <form onSubmit={addManualItem} className="bg-white p-4 rounded-2xl shadow-sm space-y-3 mb-6 border border-blue-100 animate-in slide-in-from-top-2">
                  <h3 className="font-bold text-slate-700 text-sm">Adicionar Novo Manual</h3>
                  <div className="space-y-2">
                    <div className="relative">
                      <FileText className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400" size={16} />
                      <input type="text" placeholder="Nome do Manual (ex: Esquema Elétrico)" value={newManualName} onChange={(e) => setNewManualName(e.target.value)} className="w-full pl-10 pr-3 py-2 border border-slate-200 rounded-lg text-sm focus:ring-2 focus:ring-blue-500 outline-none" />
                    </div>
                    <div className="relative">
                      <LinkIcon className="absolute left-3 top-1/2 -translate-y-1/2 text-slate-400" size={16} />
                      <input type="text" placeholder="Link do arquivo (opcional)" value={newManualUrl} onChange={(e) => setNewManualUrl(e.target.value)} className="w-full pl-10 pr-3 py-2 border border-slate-200 rounded-lg text-sm focus:ring-2 focus:ring-blue-500 outline-none" />
                    </div>
                  </div>
                  <button type="submit" className="w-full bg-blue-600 text-white p-2.5 rounded-lg font-bold text-sm hover:bg-blue-700">Adicionar à Biblioteca</button>
                </form>
              )}

              <div className="space-y-3">
                {manuals.length === 0 ? (
                  <p className="text-center text-slate-400 py-10">Nenhum manual cadastrado.</p>
                ) : (
                  manuals.map(item => (
                    <div key={item.id} className="bg-white p-4 rounded-2xl flex justify-between items-center shadow-sm border border-slate-100">
                      <div className="flex items-center gap-4">
                        <div className="w-10 h-10 bg-blue-50 text-blue-600 rounded-xl flex items-center justify-center shrink-0">
                          <FileText size={20} />
                        </div>
                        <div className="pr-2">
                          <p className="font-bold text-slate-800 text-sm leading-tight">{item.name}</p>
                          <a href={item.url} target="_blank" rel="noopener noreferrer" className="text-[10px] text-blue-500 hover:underline flex items-center gap-1 mt-1">
                            <LinkIcon size={10} /> Aceder Documento
                          </a>
                        </div>
                      </div>
                      <button onClick={() => deleteItem(item.id, 'manuals')} className="p-2 text-slate-300 hover:text-red-600 hover:bg-red-50 rounded-lg transition-colors">
                        <Trash2 size={18} />
                      </button>
                    </div>
                  ))
                )}
              </div>
            </div>
          )}
        </main>

        <nav className="fixed bottom-0 w-full max-w-md bg-white/90 backdrop-blur-md border-t p-4 flex justify-around items-center z-50 rounded-t-2xl shadow-lg">
          <button onClick={() => setCurrentView('home')} className={`flex flex-col items-center gap-1 ${currentView === 'home' ? 'text-red-600' : 'text-slate-400'}`}>
            <Home size={22} />
            <span className="text-[10px] font-bold">Início</span>
          </button>
          <button onClick={() => setCurrentView('inventory')} className={`flex flex-col items-center gap-1 ${currentView === 'inventory' ? 'text-red-600' : 'text-slate-400'}`}>
            <Package size={22} />
            <span className="text-[10px] font-bold">Estoque</span>
          </button>
          <button onClick={() => setCurrentView('tools')} className={`flex flex-col items-center gap-1 ${currentView === 'tools' ? 'text-red-600' : 'text-slate-400'}`}>
            <Wrench size={22} />
            <span className="text-[10px] font-bold">Recursos</span>
          </button>
          <button onClick={() => setCurrentView('manuals')} className={`flex flex-col items-center gap-1 ${currentView === 'manuals' ? 'text-blue-600' : 'text-slate-400'}`}>
            <FileText size={22} />
            <span className="text-[10px] font-bold">Manuais</span>
          </button>
        </nav>
      </div>
    </div>
  );
}
