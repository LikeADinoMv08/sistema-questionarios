import React, { useState, useEffect } from 'react';
import { AlertCircle, Plus, Trash2, Send, LogOut, Clock, Users, FileText, Eye, Mail, Edit2, Check, BarChart3, TrendingUp, Copy, Key } from 'lucide-react';

const QuizSystem = () => {
  const [currentView, setCurrentView] = useState('home');
  const [userType, setUserType] = useState(null);
  const [user, setUser] = useState(null);
  const [quizzes, setQuizzes] = useState(() => {
    try {
      const saved = localStorage.getItem('quizzes');
      return saved ? JSON.parse(saved) : [];
    } catch {
      return [];
    }
  });
  const [activeQuiz, setActiveQuiz] = useState(null);
  const [userAnswers, setUserAnswers] = useState({});
  const [isTabActive, setIsTabActive] = useState(true);
  const [activityLog, setActivityLog] = useState([]);
  const [editingQuiz, setEditingQuiz] = useState(null);
  const [copiedCode, setCopiedCode] = useState(false);
  const [accessCode, setAccessCode] = useState('');

  const [loginForm, setLoginForm] = useState({ email: '', password: '', name: '', accessCode: '' });
  const [newQuiz, setNewQuiz] = useState({
    title: '',
    description: '',
    questions: [{ question: '', type: 'text', options: ['', '', '', ''] }],
    duration: 60,
    endDate: '',
    active: true
  });

  const generateAccessCode = () => {
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    let code = '';
    for (let i = 0; i < 6; i++) {
      code += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return code;
  };

  useEffect(() => {
    try {
      localStorage.setItem('quizzes', JSON.stringify(quizzes));
    } catch (error) {
      console.error('Erro ao salvar questionários:', error);
    }
  }, [quizzes]);

  useEffect(() => {
    if (user && user.role === 'respondent' && activeQuiz) {
      const handleVisibilityChange = () => {
        const isHidden = document.hidden;
        setIsTabActive(!isHidden);
        
        if (isHidden) {
          logActivity('Saiu da aba ou mudou para outra aba');
          sendEmailAlert(activeQuiz.creatorEmail, `${user.name} (${user.email}) saiu da aba`);
        } else {
          logActivity('Retornou à aba');
        }
      };

      const handleBlur = () => {
        logActivity('Janela perdeu o foco');
        sendEmailAlert(activeQuiz.creatorEmail, `${user.name} (${user.email}) saiu da janela`);
      };

      document.addEventListener('visibilitychange', handleVisibilityChange);
      window.addEventListener('blur', handleBlur);

      return () => {
        document.removeEventListener('visibilitychange', handleVisibilityChange);
        window.removeEventListener('blur', handleBlur);
      };
    }
  }, [user, activeQuiz]);

  const logActivity = (action) => {
    const log = {
      timestamp: new Date().toISOString(),
      action,
      userName: user?.name,
      userEmail: user?.email,
      quiz: activeQuiz?.title
    };
    setActivityLog(prev => [...prev, log]);
  };

  const sendEmailAlert = (to, message) => {
    console.log(`Email enviado para ${to}: ${message}`);
  };

  const handleCreatorLogin = (e) => {
    e.preventDefault();
    setUser({ 
      email: loginForm.email, 
      role: 'creator',
      id: Date.now()
    });
    setCurrentView('dashboard');
  };

  const handleRespondentAccess = (e) => {
    e.preventDefault();
    
    if (!loginForm.name || !loginForm.email || !loginForm.accessCode) {
      alert('Por favor, preencha todos os campos');
      return;
    }
    
    const codeToFind = loginForm.accessCode.toUpperCase().trim();
    const quiz = quizzes.find(q => q.accessCode === codeToFind);
    
    if (!quiz) {
      alert(`Código de acesso inválido!\n\nCódigo digitado: ${codeToFind}\nQuestionários disponíveis: ${quizzes.length}\n\nVerifique se o código foi digitado corretamente.`);
      console.log('Código procurado:', codeToFind);
      console.log('Códigos disponíveis:', quizzes.map(q => q.accessCode));
      return;
    }
    
    if (!quiz.active) {
      alert('Este questionário está encerrado!');
      return;
    }
    
    setUser({ 
      email: loginForm.email,
      name: loginForm.name,
      role: 'respondent',
      id: Date.now()
    });
    
    setActiveQuiz(quiz);
    setCurrentView('answer');
    setLoginForm({ email: '', password: '', name: '', accessCode: '' });
  };

  const handleCreateQuiz = () => {
    const code = generateAccessCode();
    const quiz = {
      ...newQuiz,
      id: Date.now(),
      accessCode: code,
      creatorEmail: user.email,
      createdAt: new Date().toISOString(),
      responses: []
    };
    setQuizzes(prev => [...prev, quiz]);
    setNewQuiz({
      title: '',
      description: '',
      questions: [{ question: '', type: 'text', options: ['', '', '', ''] }],
      duration: 60,
      endDate: '',
      active: true
    });
    setCurrentView('dashboard');
    alert(`Questionário criado!\n\nCódigo de Acesso: ${code}\n\nCompartilhe este código com os respondentes.`);
  };

  const handleUpdateQuiz = () => {
    setQuizzes(prev => prev.map(q => 
      q.id === editingQuiz.id ? { ...editingQuiz } : q
    ));
    setEditingQuiz(null);
    setCurrentView('dashboard');
    alert('Questionário atualizado com sucesso!');
  };

  const startEditQuiz = (quiz) => {
    setEditingQuiz({ ...quiz });
    setCurrentView('edit');
  };

  const addQuestion = (isEditing = false) => {
    const quiz = isEditing ? editingQuiz : newQuiz;
    const setQuiz = isEditing ? setEditingQuiz : setNewQuiz;
    
    if (quiz.questions.length < 20) {
      setQuiz({
        ...quiz,
        questions: [...quiz.questions, { question: '', type: 'text', options: ['', '', '', ''] }]
      });
    } else {
      alert('Máximo de 20 perguntas atingido');
    }
  };

  const updateQuestion = (index, field, value, isEditing = false) => {
    const quiz = isEditing ? editingQuiz : newQuiz;
    const setQuiz = isEditing ? setEditingQuiz : setNewQuiz;
    
    const updated = [...quiz.questions];
    updated[index][field] = value;
    
    if (field === 'type' && value === 'multiple' && (!updated[index].options || updated[index].options.length === 0)) {
      updated[index].options = ['', '', '', ''];
    }
    
    setQuiz({ ...quiz, questions: updated });
  };

  const updateOption = (questionIndex, optionIndex, value, isEditing = false) => {
    const quiz = isEditing ? editingQuiz : newQuiz;
    const setQuiz = isEditing ? setEditingQuiz : setNewQuiz;
    
    const updated = [...quiz.questions];
    updated[questionIndex].options[optionIndex] = value;
    setQuiz({ ...quiz, questions: updated });
  };

  const addOption = (questionIndex, isEditing = false) => {
    const quiz = isEditing ? editingQuiz : newQuiz;
    const setQuiz = isEditing ? setEditingQuiz : setNewQuiz;
    
    const updated = [...quiz.questions];
    updated[questionIndex].options.push('');
    setQuiz({ ...quiz, questions: updated });
  };

  const removeOption = (questionIndex, optionIndex, isEditing = false) => {
    const quiz = isEditing ? editingQuiz : newQuiz;
    const setQuiz = isEditing ? setEditingQuiz : setNewQuiz;
    
    const updated = [...quiz.questions];
    if (updated[questionIndex].options.length > 2) {
      updated[questionIndex].options.splice(optionIndex, 1);
      setQuiz({ ...quiz, questions: updated });
    } else {
      alert('Mínimo de 2 opções necessário');
    }
  };

  const removeQuestion = (index, isEditing = false) => {
    const quiz = isEditing ? editingQuiz : newQuiz;
    const setQuiz = isEditing ? setEditingQuiz : setNewQuiz;
    
    const updated = quiz.questions.filter((_, i) => i !== index);
    setQuiz({ ...quiz, questions: updated });
  };

  const handleFileUpload = (e, isEditing = false) => {
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (event) => {
        try {
          const text = event.target.result;
          const lines = text.split('\n').filter(l => l.trim());
          const questions = lines.map(line => ({
            question: line.trim(),
            type: 'text',
            options: ['', '', '', '']
          }));
          
          const quiz = isEditing ? editingQuiz : newQuiz;
          const setQuiz = isEditing ? setEditingQuiz : setNewQuiz;
          
          setQuiz({ ...quiz, questions: questions.slice(0, 20) });
          alert(`${questions.length} perguntas importadas!`);
        } catch (error) {
          alert('Erro ao processar arquivo.');
        }
      };
      reader.readAsText(file);
    }
  };

  const copyAccessCode = (code) => {
    navigator.clipboard.writeText(code).then(() => {
      setCopiedCode(true);
      setTimeout(() => setCopiedCode(false), 2000);
      alert(`Código copiado: ${code}\n\nCompartilhe com os respondentes!`);
    }).catch(() => {
      alert(`Código de Acesso: ${code}\n\nCopie e compartilhe este código.`);
    });
  };

  const toggleQuizStatus = (quizId) => {
    setQuizzes(prev => prev.map(q => 
      q.id === quizId ? { ...q, active: !q.active } : q
    ));
  };

  const submitAnswers = () => {
    const response = {
      respondentName: user.name,
      respondentEmail: user.email,
      answers: userAnswers,
      submittedAt: new Date().toISOString(),
      activityLog: activityLog.filter(log => log.quiz === activeQuiz.title)
    };
    
    setQuizzes(prev => prev.map(q => 
      q.id === activeQuiz.id 
        ? { ...q, responses: [...(q.responses || []), response] }
        : q
    ));
    
    sendEmailAlert(activeQuiz.creatorEmail, `${user.name} completou o questionário`);
    alert('Respostas enviadas com sucesso! Obrigado por participar.');
    setActiveQuiz(null);
    setUserAnswers({});
    setActivityLog([]);
    setLoginForm({ email: '', password: '', name: '', accessCode: '' });
    setUser(null);
    setCurrentView('home');
  };

  const getQuizStatistics = (quiz) => {
    const totalResponses = quiz.responses?.length || 0;
    const questionsStats = quiz.questions.map((q, index) => {
      if (q.type === 'multiple') {
        const answerCounts = {};
        q.options.forEach(opt => answerCounts[opt] = 0);
        
        quiz.responses?.forEach(response => {
          const answer = response.answers[index];
          if (answer && answerCounts[answer] !== undefined) {
            answerCounts[answer]++;
          }
        });
        
        return { question: q.question, type: 'multiple', answerCounts };
      }
      return { question: q.question, type: q.type };
    });
    
    return { totalResponses, questionsStats };
  };

  const renderHome = () => (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 flex items-center justify-center p-4">
      <div className="bg-white rounded-2xl shadow-xl p-8 w-full max-w-2xl">
        <div className="text-center mb-8">
          <FileText className="w-20 h-20 text-indigo-600 mx-auto mb-4" />
          <h1 className="text-4xl font-bold text-gray-900 mb-2">Sistema de Questionários</h1>
          <p className="text-gray-600">Escolha como deseja acessar</p>
        </div>
        
        <div className="grid md:grid-cols-2 gap-6">
          <button
            onClick={() => {
              setUserType('creator');
              setCurrentView('creatorLogin');
            }}
            className="bg-indigo-600 text-white p-8 rounded-xl hover:bg-indigo-700 transition flex flex-col items-center space-y-4"
          >
            <Edit2 className="w-16 h-16" />
            <div className="text-center">
              <h3 className="text-xl font-bold mb-2">Criador</h3>
              <p className="text-indigo-100 text-sm">Criar e gerenciar questionários</p>
            </div>
          </button>
          
          <button
            onClick={() => {
              setUserType('respondent');
              setCurrentView('respondentLogin');
            }}
            className="bg-green-600 text-white p-8 rounded-xl hover:bg-green-700 transition flex flex-col items-center space-y-4"
          >
            <Users className="w-16 h-16" />
            <div className="text-center">
              <h3 className="text-xl font-bold mb-2">Respondente</h3>
              <p className="text-green-100 text-sm">Responder questionários</p>
            </div>
          </button>
        </div>
      </div>
    </div>
  );

  const renderCreatorLogin = () => (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 flex items-center justify-center p-4">
      <div className="bg-white rounded-2xl shadow-xl p-8 w-full max-w-md">
        <button
          onClick={() => setCurrentView('home')}
          className="mb-6 text-indigo-600 hover:text-indigo-700"
        >
          ← Voltar
        </button>
        
        <div className="text-center mb-8">
          <Edit2 className="w-16 h-16 text-indigo-600 mx-auto mb-4" />
          <h1 className="text-3xl font-bold text-gray-900">Área do Criador</h1>
          <p className="text-gray-600 mt-2">Faça login para gerenciar</p>
        </div>
        
        <div className="space-y-4">
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">Email</label>
            <input
              type="email"
              className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
              value={loginForm.email}
              onChange={(e) => setLoginForm({ ...loginForm, email: e.target.value })}
              placeholder="criador@email.com"
            />
          </div>
          
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-2">Senha</label>
            <input
              type="password"
              className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
              value={loginForm.password}
              onChange={(e) => setLoginForm({ ...loginForm, password: e.target.value })}
              placeholder="••••••••"
            />
          </div>
          
          <button
            onClick={handleCreatorLogin}
            className="w-full bg-indigo-600 text-white py-3 rounded-lg font-semibold hover:bg-indigo-700"
          >
            Entrar como Criador
          </button>
        </div>
      </div>
    </div>
  );

  const renderRespondentLogin = () => {
    const activeQuizzes = quizzes.filter(q => q.active);
    
    return (
      <div className="min-h-screen bg-gradient-to-br from-green-50 to-emerald-100 flex items-center justify-center p-4">
        <div className="bg-white rounded-2xl shadow-xl p-8 w-full max-w-md">
          <button
            onClick={() => setCurrentView('home')}
            className="mb-6 text-green-600 hover:text-green-700"
          >
            ← Voltar
          </button>
          
          <div className="text-center mb-8">
            <Users className="w-16 h-16 text-green-600 mx-auto mb-4" />
            <h1 className="text-3xl font-bold text-gray-900">Área do Respondente</h1>
            <p className="text-gray-600 mt-2">Preencha seus dados e o código de acesso</p>
          </div>
          
          {activeQuizzes.length > 0 ? (
            <div className="mb-6 p-4 bg-blue-50 border-2 border-blue-300 rounded-lg">
              <p className="text-sm font-bold text-blue-900 mb-3 flex items-center">
                <FileText className="w-4 h-4 mr-2" />
                Questionários Disponíveis ({activeQuizzes.length})
              </p>
              <div className="space-y-2">
                {activeQuizzes.map(q => (
                  <div key={q.id} className="bg-white p-3 rounded-lg border border-blue-200">
                    <div className="flex justify-between items-center">
                      <div className="flex-1">
                        <p className="text-sm font-medium text-gray-900">{q.title}</p>
                        <p className="text-xs text-gray-500">{q.questions.length} perguntas</p>
                      </div>
                      <div className="ml-3 flex items-center space-x-2">
                        <Key className="w-4 h-4 text-blue-600" />
                        <span className="font-mono font-bold text-lg text-blue-900 bg-blue-100 px-3 py-1 rounded">
                          {q.accessCode}
                        </span>
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          ) : (
            <div className="mb-6 p-4 bg-yellow-50 border-2 border-yellow-300 rounded-lg">
              <p className="text-sm font-medium text-yellow-900 flex items-center">
                <AlertCircle className="w-4 h-4 mr-2" />
                Nenhum questionário disponível no momento
              </p>
              <p className="text-xs text-yellow-700 mt-1">Aguarde um criador gerar um questionário</p>
            </div>
          )}
          
          <div className="space-y-4">
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-2">Nome Completo *</label>
              <input
                type="text"
                required
                className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500"
                value={loginForm.name}
                onChange={(e) => setLoginForm({ ...loginForm, name: e.target.value })}
                placeholder="Seu nome completo"
              />
            </div>
            
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-2">Email *</label>
              <input
                type="email"
                required
                className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500"
                value={loginForm.email}
                onChange={(e) => setLoginForm({ ...loginForm, email: e.target.value })}
                placeholder="seu@email.com"
              />
            </div>
            
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-2">Código de Acesso *</label>
              <div className="relative">
                <Key className="absolute left-3 top-1/2 transform -translate-y-1/2 w-5 h-5 text-gray-400" />
                <input
                  type="text"
                  required
                  maxLength="6"
                  className="w-full pl-10 pr-4 py-3 border-2 border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500 focus:border-green-500 uppercase font-mono text-xl tracking-widest text-center"
                  value={loginForm.accessCode}
                  onChange={(e) => setLoginForm({ ...loginForm, accessCode: e.target.value.toUpperCase() })}
                  placeholder="ABC123"
                />
              </div>
              <p className="text-xs text-gray-500 mt-1 text-center">Digite exatamente como mostrado acima</p>
            </div>
            
            <button
              onClick={handleRespondentAccess}
              disabled={activeQuizzes.length === 0}
              className="w-full bg-green-600 text-white py-3 rounded-lg font-semibold hover:bg-green-700 disabled:bg-gray-400 disabled:cursor-not-allowed transition"
            >
              Acessar Questionário
            </button>
          </div>
        </div>
      </div>
    );
  };

  const renderDashboard = () => (
    <div className="min-h-screen bg-gray-50">
      <nav className="bg-indigo-600 text-white shadow-lg">
        <div className="max-w-7xl mx-auto px-4 py-4 flex justify-between items-center">
          <div className="flex items-center space-x-3">
            <Edit2 className="w-8 h-8" />
            <div>
              <h1 className="text-xl font-bold">Painel do Criador</h1>
              <p className="text-sm text-indigo-100">{user.email}</p>
            </div>
          </div>
          <button
            onClick={() => {
              setUser(null);
              setCurrentView('home');
            }}
            className="flex items-center space-x-2 bg-indigo-700 hover:bg-indigo-800 px-4 py-2 rounded-lg"
          >
            <LogOut className="w-5 h-5" />
            <span>Sair</span>
          </button>
        </div>
      </nav>

      <div className="max-w-7xl mx-auto px-4 py-8">
        <div className="mb-6">
          <button
            onClick={() => setCurrentView('create')}
            className="bg-indigo-600 text-white px-6 py-3 rounded-lg font-semibold hover:bg-indigo-700 flex items-center space-x-2"
          >
            <Plus className="w-5 h-5" />
            <span>Criar Novo Questionário</span>
          </button>
        </div>

        <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
          {quizzes
            .filter(q => q.creatorEmail === user.email)
            .map(quiz => {
              const stats = getQuizStatistics(quiz);
              return (
                <div key={quiz.id} className="bg-white rounded-xl shadow-md p-6 hover:shadow-lg transition">
                  <div className="flex justify-between items-start mb-4">
                    <h3 className="text-xl font-bold text-gray-900">{quiz.title}</h3>
                    <span className={`px-3 py-1 rounded-full text-xs font-semibold ${quiz.active ? 'bg-green-100 text-green-800' : 'bg-red-100 text-red-800'}`}>
                      {quiz.active ? 'Ativo' : 'Encerrado'}
                    </span>
                  </div>
                  
                  <p className="text-gray-600 mb-4 line-clamp-2">{quiz.description}</p>
                  
                  <div className="bg-indigo-50 border-2 border-indigo-200 rounded-lg p-3 mb-4">
                    <div className="flex items-center justify-between">
                      <div>
                        <p className="text-xs text-indigo-600 font-medium mb-1">Código de Acesso</p>
                        <p className="text-2xl font-bold text-indigo-900 font-mono tracking-widest">{quiz.accessCode}</p>
                      </div>
                      <button
                        onClick={() => copyAccessCode(quiz.accessCode)}
                        className="bg-indigo-600 text-white p-2 rounded-lg hover:bg-indigo-700"
                      >
                        {copiedCode ? <Check className="w-5 h-5" /> : <Copy className="w-5 h-5" />}
                      </button>
                    </div>
                  </div>
                  
                  <div className="space-y-2 text-sm text-gray-500 mb-4">
                    <div className="flex items-center space-x-2">
                      <Clock className="w-4 h-4" />
                      <span>{quiz.duration} min</span>
                    </div>
                    <div className="flex items-center space-x-2">
                      <FileText className="w-4 h-4" />
                      <span>{quiz.questions.length} perguntas</span>
                    </div>
                    <div className="flex items-center space-x-2">
                      <Users className="w-4 h-4 text-indigo-600" />
                      <span className="text-indigo-600 font-medium">{stats.totalResponses} respostas</span>
                    </div>
                  </div>

                  <div className="space-y-2">
                    <button
                      onClick={() => startEditQuiz(quiz)}
                      className="w-full bg-amber-50 text-amber-700 px-4 py-2 rounded-lg font-medium hover:bg-amber-100 flex items-center justify-center space-x-2"
                    >
                      <Edit2 className="w-4 h-4" />
                      <span>Editar</span>
                    </button>
                    <button
                      onClick={() => toggleQuizStatus(quiz.id)}
                      className="w-full bg-gray-100 text-gray-700 px-4 py-2 rounded-lg font-medium hover:bg-gray-200"
                    >
                      {quiz.active ? 'Encerrar' : 'Reativar'}
                    </button>
                    <button
                      onClick={() => {
                        setActiveQuiz(quiz);
                        setCurrentView('realTimeResults');
                      }}
                      className="w-full bg-indigo-600 text-white px-4 py-2 rounded-lg font-medium hover:bg-indigo-700 flex items-center justify-center space-x-2"
                    >
                      <BarChart3 className="w-4 h-4" />
                      <span>Resultados</span>
                    </button>
                  </div>
                </div>
              );
            })}
        </div>

        {quizzes.filter(q => q.creatorEmail === user.email).length === 0 && (
          <div className="text-center py-12">
            <FileText className="w-16 h-16 text-gray-400 mx-auto mb-4" />
            <p className="text-gray-600">Nenhum questionário criado</p>
          </div>
        )}
      </div>
    </div>
  );

  const renderQuestionForm = (quiz, setQuiz, isEditing) => (
    <div className="space-y-6">
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-2">Título</label>
        <input
          type="text"
          className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
          value={quiz.title}
          onChange={(e) => setQuiz({ ...quiz, title: e.target.value })}
        />
      </div>

      <div>
        <label className="block text-sm font-medium text-gray-700 mb-2">Descrição</label>
        <textarea
          className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
          value={quiz.description}
          onChange={(e) => setQuiz({ ...quiz, description: e.target.value })}
          rows="3"
        />
      </div>

      <div className="grid grid-cols-2 gap-4">
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-2">Duração (min)</label>
          <input
            type="number"
            className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
            value={quiz.duration}
            onChange={(e) => setQuiz({ ...quiz, duration: parseInt(e.target.value) })}
            min="1"
          />
        </div>

        <div>
          <label className="block text-sm font-medium text-gray-700 mb-2">Encerramento</label>
          <input
            type="datetime-local"
            className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500"
            value={quiz.endDate}
            onChange={(e) => setQuiz({ ...quiz, endDate: e.target.value })}
          />
        </div>
      </div>

      <div>
        <label className="block text-sm font-medium text-gray-700 mb-2">Importar (txt)</label>
        <input
          type="file"
          accept=".txt"
          onChange={(e) => handleFileUpload(e, isEditing)}
          className="w-full px-4 py-2 border border-gray-300 rounded-lg"
        />
      </div>

      <div className="border-t pt-6">
        <div className="flex justify-between items-center mb-4">
          <h3 className="text-lg font-semibold">Perguntas ({quiz.questions.length}/20)</h3>
          <button
            onClick={() => addQuestion(isEditing)}
            disabled={quiz.questions.length >= 20}
            className="flex items-center space-x-2 text-indigo-600 hover:text-indigo-700 disabled:opacity-50"
          >
            <Plus className="w-5 h-5" />
            <span>Adicionar</span>
          </button>
        </div>

        <div className="space-y-4 max-h-96 overflow-y-auto">
          {quiz.questions.map((q, idx) => (
            <div key={idx} className="bg-gray-50 p-4 rounded-lg border">
              <div className="flex justify-between mb-3">
                <span className="text-sm font-medium">Pergunta {idx + 1}</span>
                <button
                  onClick={() => removeQuestion(idx, isEditing)}
                  className="text-red-600 hover:text-red-700"
                >
                  <Trash2 className="w-4 h-4" />
                </button>
              </div>
              
              <input
                type="text"
                className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 mb-2"
                value={q.question}
                onChange={(e) => updateQuestion(idx, 'question', e.target.value, isEditing)}
                placeholder="Digite a pergunta"
              />
              
              <select
                className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 mb-3"
                value={q.type}
                onChange={(e) => updateQuestion(idx, 'type', e.target.value, isEditing)}
              >
                <option value="text">Resposta Curta</option>
                <option value="textarea">Resposta Longa</option>
                <option value="multiple">Múltipla Escolha</option>
              </select>

              {q.type === 'multiple' && (
                <div className="bg-white p-3 rounded-lg border">
                  <div className="flex justify-between mb-2">
                    <span className="text-sm font-medium">Alternativas</span>
                    <button
                      onClick={() => addOption(idx, isEditing)}
                      className="text-indigo-600 hover:text-indigo-700 text-sm flex items-center space-x-1"
                    >
                      <Plus className="w-4 h-4" />
                      <span>Adicionar</span>
                    </button>
                  </div>
                  <div className="space-y-2">
                    {q.options.map((opt, optIdx) => (
                      <div key={optIdx} className="flex items-center space-x-2">
                        <span className="text-sm text-gray-500 w-6">{String.fromCharCode(65 + optIdx)})</span>
                        <input
                          type="text"
                          className="flex-1 px-3 py-2 border rounded-lg focus:ring-2 focus:ring-indigo-500 text-sm"
                          value={opt}
                          onChange={(e) => updateOption(idx, optIdx, e.target.value, isEditing)}
                          placeholder={`Alternativa ${String.fromCharCode(65 + optIdx)}`}
                        />
                        <button
                          onClick={() => removeOption(idx, optIdx, isEditing)}
                          className="text-red-600 hover:text-red-700"
                        >
                          <Trash2 className="w-4 h-4" />
                        </button>
                      </div>
                    ))}
                  </div>
                </div>
              )}
            </div>
          ))}
        </div>
      </div>
    </div>
  );

  const renderCreateQuiz = () => (
    <div className="min-h-screen bg-gray-50 py-8">
      <div className="max-w-4xl mx-auto px-4">
        <button
          onClick={() => setCurrentView('dashboard')}
          className="mb-6 text-indigo-600 hover:text-indigo-700"
        >
          ← Voltar
        </button>

        <div className="bg-white rounded-xl shadow-md p-8">
          <h2 className="text-2xl font-bold text-gray-900 mb-6">Criar Novo Questionário</h2>
          {renderQuestionForm(newQuiz, setNewQuiz, false)}
          <button
            onClick={handleCreateQuiz}
            disabled={!newQuiz.title || newQuiz.questions.length === 0}
            className="w-full bg-indigo-600 text-white py-3 rounded-lg font-semibold hover:bg-indigo-700 disabled:opacity-50 mt-6"
          >
            Criar Questionário
          </button>
        </div>
      </div>
    </div>
  );

  const renderEditQuiz = () => (
    <div className="min-h-screen bg-gray-50 py-8">
      <div className="max-w-4xl mx-auto px-4">
        <button
          onClick={() => {
            setEditingQuiz(null);
            setCurrentView('dashboard');
          }}
          className="mb-6 text-indigo-600 hover:text-indigo-700"
        >
          ← Voltar
        </button>

        <div className="bg-white rounded-xl shadow-md p-8">
          <h2 className="text-2xl font-bold text-gray-900 mb-6">Editar Questionário</h2>
          {renderQuestionForm(editingQuiz, setEditingQuiz, true)}
          <button
            onClick={handleUpdateQuiz}
            disabled={!editingQuiz.title || editingQuiz.questions.length === 0}
            className="w-full bg-indigo-600 text-white py-3 rounded-lg font-semibold hover:bg-indigo-700 disabled:opacity-50 mt-6"
          >
            Salvar Alterações
          </button>
        </div>
      </div>
    </div>
  );

  const renderAnswerQuiz = () => (
    <div className="min-h-screen bg-gray-50 py-8">
      <div className="max-w-3xl mx-auto px-4">
        <div className="bg-white rounded-xl shadow-md p-8">
          <div className="mb-6 flex items-center justify-between">
            <div>
              <h2 className="text-3xl font-bold text-gray-900 mb-2">{activeQuiz?.title}</h2>
              <p className="text-gray-600">{activeQuiz?.description}</p>
            </div>
            <div className="text-right">
              <p className="text-sm text-gray-500">Respondente:</p>
              <p className="font-medium text-gray-900">{user?.name}</p>
              <p className="text-sm text-gray-600">{user?.email}</p>
            </div>
          </div>
          
          {!isTabActive && (
            <div className="bg-red-50 border border-red-200 rounded-lg p-4 flex items-start space-x-3 mb-6">
              <AlertCircle className="w-5 h-5 text-red-600 flex-shrink-0 mt-0.5" />
              <div>
                <p className="text-red-800 font-medium">Atenção!</p>
                <p className="text-red-700 text-sm">Detectamos que você saiu da aba. O criador foi notificado.</p>
              </div>
            </div>
          )}

          <div className="space-y-6">
            {activeQuiz?.questions.map((q, idx) => (
              <div key={idx} className="border-b pb-6 last:border-b-0">
                <label className="block text-lg font-medium text-gray-900 mb-3">
                  {idx + 1}. {q.question}
                </label>
                
                {q.type === 'text' && (
                  <input
                    type="text"
                    className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-green-500"
                    value={userAnswers[idx] || ''}
                    onChange={(e) => setUserAnswers({ ...userAnswers, [idx]: e.target.value })}
                    placeholder="Sua resposta"
                  />
                )}
                
                {q.type === 'textarea' && (
                  <textarea
                    className="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-green-500"
                    value={userAnswers[idx] || ''}
                    onChange={(e) => setUserAnswers({ ...userAnswers, [idx]: e.target.value })}
                    placeholder="Sua resposta"
                    rows="4"
                  />
                )}
                
                {q.type === 'multiple' && (
                  <div className="space-y-2">
                    {q.options.filter(opt => opt.trim()).map((opt, optIdx) => (
                      <label key={optIdx} className="flex items-center space-x-3 p-3 border rounded-lg hover:bg-green-50 cursor-pointer">
                        <input
                          type="radio"
                          name={`question-${idx}`}
                          value={opt}
                          checked={userAnswers[idx] === opt}
                          onChange={(e) => setUserAnswers({ ...userAnswers, [idx]: e.target.value })}
                          className="w-4 h-4 text-green-600"
                        />
                        <span className="text-gray-700">{String.fromCharCode(65 + optIdx)}) {opt}</span>
                      </label>
                    ))}
                  </div>
                )}
              </div>
            ))}

            <button
              onClick={submitAnswers}
              className="w-full bg-green-600 text-white py-3 rounded-lg font-semibold hover:bg-green-700 flex items-center justify-center space-x-2"
            >
              <Send className="w-5 h-5" />
              <span>Enviar Respostas</span>
            </button>
          </div>
        </div>
      </div>
    </div>
  );

  const renderRealTimeResults = () => {
    const stats = getQuizStatistics(activeQuiz);
    
    return (
      <div className="min-h-screen bg-gray-50 py-8">
        <div className="max-w-6xl mx-auto px-4">
          <button
            onClick={() => {
              setActiveQuiz(null);
              setCurrentView('dashboard');
            }}
            className="mb-6 text-indigo-600 hover:text-indigo-700"
          >
            ← Voltar
          </button>

          <div className="bg-white rounded-xl shadow-md p-8">
            <div className="flex justify-between items-center mb-6">
              <div>
                <h2 className="text-2xl font-bold text-gray-900">{activeQuiz?.title}</h2>
                <p className="text-gray-600">Resultados em Tempo Real</p>
              </div>
              <div className="text-right">
                <div className="flex items-center space-x-2 bg-indigo-100 text-indigo-800 px-4 py-2 rounded-lg mb-2">
                  <Key className="w-5 h-5" />
                  <span className="font-mono font-bold text-xl">{activeQuiz?.accessCode}</span>
                </div>
                <div className="flex items-center space-x-2 bg-green-100 text-green-800 px-4 py-2 rounded-lg">
                  <TrendingUp className="w-5 h-5" />
                  <span className="font-semibold">Atualizado</span>
                </div>
              </div>
            </div>
            
            <div className="grid md:grid-cols-3 gap-6 mb-8">
              <div className="bg-indigo-50 p-6 rounded-xl">
                <div className="flex items-center space-x-3 mb-2">
                  <Users className="w-8 h-8 text-indigo-600" />
                  <span className="text-sm font-medium text-indigo-900">Total Respostas</span>
                </div>
                <p className="text-4xl font-bold text-indigo-600">{stats.totalResponses}</p>
              </div>
              
              <div className="bg-blue-50 p-6 rounded-xl">
                <div className="flex items-center space-x-3 mb-2">
                  <FileText className="w-8 h-8 text-blue-600" />
                  <span className="text-sm font-medium text-blue-900">Perguntas</span>
                </div>
                <p className="text-4xl font-bold text-blue-600">{activeQuiz.questions.length}</p>
              </div>
              
              <div className="bg-green-50 p-6 rounded-xl">
                <div className="flex items-center space-x-3 mb-2">
                  <BarChart3 className="w-8 h-8 text-green-600" />
                  <span className="text-sm font-medium text-green-900">Status</span>
                </div>
                <p className="text-2xl font-bold text-green-600">
                  {activeQuiz.active ? 'Ativo' : 'Encerrado'}
                </p>
              </div>
            </div>

            <div className="space-y-8">
              <h3 className="text-xl font-bold text-gray-900">Análise por Pergunta</h3>
              
              {stats.questionsStats.map((stat, idx) => (
                <div key={idx} className="border rounded-lg p-6">
                  <h4 className="font-semibold text-gray-900 mb-4">
                    {idx + 1}. {stat.question}
                  </h4>
                  
                  {stat.type === 'multiple' && stat.answerCounts && (
                    <div className="space-y-3">
                      {Object.entries(stat.answerCounts).map(([opt, count], optIdx) => {
                        const percentage = stats.totalResponses > 0 
                          ? ((count / stats.totalResponses) * 100).toFixed(1) 
                          : 0;
                        
                        return opt.trim() ? (
                          <div key={optIdx} className="space-y-1">
                            <div className="flex justify-between text-sm">
                              <span className="font-medium text-gray-700">
                                {String.fromCharCode(65 + optIdx)}) {opt}
                              </span>
                              <span className="text-gray-600">
                                {count} ({percentage}%)
                              </span>
                            </div>
                            <div className="w-full bg-gray-200 rounded-full h-3">
                              <div
                                className="bg-indigo-600 h-3 rounded-full transition-all duration-500"
                                style={{ width: `${percentage}%` }}
                              />
                            </div>
                          </div>
                        ) : null;
                      })}
                    </div>
                  )}
                  
                  {stat.type !== 'multiple' && (
                    <p className="text-gray-600 text-sm">
                      Pergunta aberta - {stats.totalResponses} resposta(s)
                    </p>
                  )}
                </div>
              ))}
            </div>

            <div className="mt-8 border-t pt-8">
              <h3 className="text-xl font-bold text-gray-900 mb-6">Respostas Individuais</h3>
              
              {activeQuiz?.responses?.length === 0 ? (
                <div className="text-center py-12">
                  <Users className="w-16 h-16 text-gray-400 mx-auto mb-4" />
                  <p className="text-gray-600">Aguardando respostas...</p>
                </div>
              ) : (
                <div className="space-y-6">
                  {activeQuiz?.responses?.map((response, idx) => (
                    <div key={idx} className="border rounded-lg p-6 bg-gray-50">
                      <div className="flex items-center justify-between mb-4">
                        <div>
                          <div className="flex items-center space-x-3 mb-1">
                            <Users className="w-5 h-5 text-gray-400" />
                            <span className="font-bold text-gray-900">{response.respondentName}</span>
                          </div>
                          <div className="flex items-center space-x-3 ml-8">
                            <Mail className="w-4 h-4 text-gray-400" />
                            <span className="text-sm text-gray-600">{response.respondentEmail}</span>
                          </div>
                        </div>
                        <span className="text-sm text-gray-500">
                          {new Date(response.submittedAt).toLocaleString('pt-BR')}
                        </span>
                      </div>

                      {response.activityLog?.length > 0 && (
                        <div className="mb-4 bg-yellow-50 border border-yellow-200 rounded-lg p-3">
                          <p className="text-sm font-medium text-yellow-800 mb-2">⚠️ Atividade Suspeita:</p>
                          <ul className="text-sm text-yellow-700 space-y-1">
                            {response.activityLog.map((log, i) => (
                              <li key={i}>• {log.action} - {new Date(log.timestamp).toLocaleTimeString('pt-BR')}</li>
                            ))}
                          </ul>
                        </div>
                      )}

                      <div className="space-y-3">
                        {Object.entries(response.answers).map(([qIdx, answer]) => (
                          <div key={qIdx} className="bg-white p-3 rounded border">
                            <p className="text-sm font-medium text-gray-700 mb-1">
                              {activeQuiz.questions[qIdx]?.question}
                            </p>
                            <p className="text-gray-900">{answer}</p>
                          </div>
                        ))}
                      </div>
                    </div>
                  ))}
                </div>
              )}
            </div>
          </div>
        </div>
      </div>
    );
  };

  return (
    <>
      {currentView === 'home' && renderHome()}
      {currentView === 'creatorLogin' && renderCreatorLogin()}
      {currentView === 'respondentLogin' && renderRespondentLogin()}
      {currentView === 'dashboard' && renderDashboard()}
      {currentView === 'create' && renderCreateQuiz()}
      {currentView === 'edit' && renderEditQuiz()}
      {currentView === 'answer' && renderAnswerQuiz()}
      {currentView === 'realTimeResults' && renderRealTimeResults()}
    </>
  );
};

export default QuizSystem;
