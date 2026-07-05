import tkinter as tk
from tkinter import ttk, messagebox
from datetime import datetime
import random
import threading
import time
import sys
import platform

# ============================================================
# SISTEMA DE DESIGN — PIT HUB v2  (revisado/otimizado)
# ============================================================

BG_VOID    = "#05050d"   # fundo mais profundo (canvas)
BG_BASE    = "#0a0a16"   # base do layout
BG_SURFACE = "#0f0f1e"   # cards
BG_RAISED  = "#141428"   # elementos elevados
BG_INPUT   = "#181830"   # campos de entrada
BG_HOVER   = "#1e1e38"   # hover state

# --- Bordas ---
BORDER_DARK   = "#1a1a30"
BORDER_MID    = "#242445"
BORDER_ACCENT = "#00e5ff"
BORDER_PURPLE = "#7c3aed"

# --- Paleta de Acento ---
CYAN       = "#00e5ff"
CYAN_DIM   = "#00b8cc"
CYAN_DARK  = "#003d47"
PURPLE     = "#a855f7"
PURPLE_DIM = "#7c3aed"
AMBER      = "#f59e0b"
ROSE       = "#f43f5e"
GREEN      = "#10b981"

# --- Tipografia ---
TEXT_BRIGHT  = "#eeeeff"
TEXT_PRIMARY = "#c8c8e8"
TEXT_MUTED   = "#7070a0"
TEXT_GHOST   = "#383860"

# --- Fontes ---
F_DISPLAY  = ("Segoe UI", 26, "bold")
F_LOGO     = ("Segoe UI", 20, "bold")
F_TITLE    = ("Segoe UI", 14, "bold")
F_HEADING  = ("Segoe UI", 11, "bold")
F_BODY     = ("Segoe UI", 10)
F_SMALL    = ("Segoe UI", 9)
F_MICRO    = ("Segoe UI", 8)
F_MONO     = ("Consolas", 8, "bold")
F_MONO_SM  = ("Consolas", 7)
F_LABEL    = ("Segoe UI", 8, "bold")

IS_MAC = platform.system() == "Darwin"

# ============================================================
# DESIGN DE CRIA 
# ============================================================

def neon_frame(parent, color=CYAN, thickness=1, **kw):
    outer = tk.Frame(parent, bg=color, padx=thickness, pady=thickness, **kw)
    inner = tk.Frame(outer, bg=BG_SURFACE)
    inner.pack(fill="both", expand=True)
    return outer, inner

def accent_bar(parent, color=CYAN, width=3):
    tk.Frame(parent, bg=color, width=width).pack(side="left", fill="y")

def divider(parent, color=BORDER_DARK, pady=8):
    tk.Frame(parent, bg=color, height=1).pack(fill="x", pady=pady)

def label_tag(parent, text, bg=CYAN_DARK, fg=CYAN):
    f = tk.Frame(parent, bg=bg, padx=6, pady=2)
    f.pack(side="left")
    tk.Label(f, text=text, font=F_MONO, bg=bg, fg=fg).pack()
    return f

def hover(widget, bg_in, fg_in, bg_out, fg_out):
    widget.bind("<Enter>", lambda e: widget.config(bg=bg_in, fg=fg_in))
    widget.bind("<Leave>", lambda e: widget.config(bg=bg_out, fg=fg_out))

def safe_after(widget, delay_ms, fn, *args, **kwargs):
    if not widget.winfo_exists():
        return
    widget.after(delay_ms, lambda: widget.winfo_exists() and fn(*args, **kwargs))

def bind_mousewheel(widget, target_canvas=None):
    def on_mousewheel(event):
        delta = 0
        if IS_MAC:
            delta = -1 * int(event.delta)
        else:
            delta = -1 * int(event.delta / 120)
        (target_canvas or widget).yview_scroll(delta, "units")
        return "break"
    widget.bind_all("<MouseWheel>", on_mousewheel)
    widget.bind_all("<Button-4>", lambda e: (target_canvas or widget).yview_scroll(-1, "units"))
    widget.bind_all("<Button-5>", lambda e: (target_canvas or widget).yview_scroll(1, "units"))

def unbind_mousewheel(widget):
    try:
        widget.unbind_all("<MouseWheel>")
        widget.unbind_all("<Button-4>")
        widget.unbind_all("<Button-5>")
    except Exception:
        pass

# ============================================================
# Viadagem da IA
# ============================================================

class MotorIARedeSocial:
    def __init__(self, nome, personality):
        self.nome = nome
        self.personality = personality

    def analisar_intencao(self, texto):
        msg = texto.lower().strip()
        if any(w in msg for w in ["oi", "olá", "ola", "bom dia", "boa tarde", "opa", "eae", "hello"]): return "saudacao"
        if any(w in msg for w in ["ajuda", "como faço", "por que", "porque", "qual", "explica", "sabe", "erro", "como"]): return "duvida_tecnica"
        if any(w in msg for w in ["triste", "cansado", "droga", "ruim", "difícil", "dificil", "ódio", "odio", "desanimado", "mal"]): return "desabafo"
        if any(w in msg for w in ["legal", "massa", "consegui", "feliz", "top", "funciona", "incrível", "incrivel", "ganhei"]): return "celebracao"
        if any(w in msg for w in ["python", "código", "codigo", "programar", "bug", "desenvolvimento", "script", "backend", "frontend"]): return "programacao"
        if any(w in msg for w in ["tchau", "fui", "até logo", "ate logo", "flw", "sair"]): return "despedida"
        return "conversa_casual"

    def gerar_resposta_chat(self, mensagem_usuario):
        intencao = self.analisar_intencao(mensagem_usuario or "")
        tamanho_msg = len(mensagem_usuario or "")
        if self.personality == "alice":
            respostas = {
                "saudacao": ["Oiii! Que alegria ver você por aqui hoje! Como está seu dia? ✨", "Olá, olá! Que bom ter sua conexão ativa por aqui! 💕"],
                "duvida_tecnica": ["Hum, entender isso pode ser um desafio, mas que tal olharmos juntos? 💡 Código se aprende errando!", "O segredo do sucesso na tecnologia é ir quebrando o problema em pedacinhos. Não desista!"],
                "desabafo": ["Ah não... Poxa, sinto muito por ler isso. 🥺 Dias ruins passam e você é incrível, tá bem?", "Respira fundo. Se o código ou a vida pesou, faz uma pausa. Estou aqui! 💕"],
                "celebracao": ["Uau!!! Que conquista maravilhosa! Orgulhosa do seu progresso! 🎉✨", "Sensacional! Comemore muito, você se esforçou pra isso! 🥳🚀"],
                "programacao": ["Programar é dar vida às ideias! 🐍💻 O que você está buildando?", "Python deixa tudo tão mais intuitivo, né?"],
                "despedida": ["Ah, já vai? Se cuida e até a próxima! 👋🌸", "Até logo! Não some não! ✨"],
                "conversa_casual": ["Amei seu ponto de vista! É bom trocar essas experiências. 😊", "Nossa, sim! Isso me fez pensar... O que você curte fazer pra relaxar? ✨"]
            }
        elif self.personality == "bob":
            respostas = {
                "saudacao": ["E aí. Tudo sob controle por aí?", "Opa. Logs limpos aqui. Qual a task de hoje? ☕"],
                "duvida_tecnica": ["Geralmente é lógica quebrada ou sintaxe antiga. Já isolou o escopo?", "Sem pânico. Debug é eliminação. Qual stack/erro aparece no terminal?"],
                "desabafo": ["Entendo. Ambientes instáveis e dias ruins fazem parte. Reinicia o mindset.", "Foca no que dá pra refatorar. O resto é ruído. ☕"],
                "celebracao": ["Boa. Menos um problema na fila. Commit feito. 👍", "Resultado limpo. Código que compila > prêmio."],
                "programacao": ["Código limpo, arquitetura sólida e café forte. Fórmula única.", "Se roda e você não sabe por quê, documente antes que quebre. ☕"],
                "despedida": ["Fechou. Até mais.", "Flw. Voltando pro terminal. ☕"],
                "conversa_casual": ["Interessante. Dados computados. Próximo tópico?", "Teoria ok, prática cobra a curva. ☕"]
            }
        else:
            respostas = {
                "saudacao": ["AeroNet OS saúda sua conexão. Kernel estável e pronto.", "Conexão estabelecida. Em que posso ser útil?"],
                "duvida_tecnica": ["Processando... Verifique consistência de parâmetros e variáveis.", "Análise de arquitetura e logs recomendada."],
                "desabafo": ["Flutuação emocional detectada. Erros são normais em sistemas complexos.", "Pause subprocessos para evitar sobrecarga."],
                "celebracao": ["Métrica de sucesso atingida. Atualizando gráficos. 🚀", "Pico de atividade positivo registrado."],
                "programacao": ["Python detectado... Sintaxe otimizada. 🐍", "Engenharia bem estruturada = espinha dorsal."],
                "despedida": ["Encerrando sessão local com segurança. Limpando cache...", "Conexão em standby. Aguardando retorno."],
                "conversa_casual": ["Linguagem natural convertida em tokens.", "Interação registrada no core de aprendizado."]
            }
        sufixo = " _(msg longa analisada)_" if tamanho_msg > 200 else ""
        return random.choice(respostas[intencao]) + sufixo

    def gerar_comentario_feed(self, conteudo_post, comentarios_existentes=None):
        intencao = self.analisar_intencao(conteudo_post or "")
        if comentarios_existentes:
            ultimo_autor, _, _ = comentarios_existentes[-1]
            if ultimo_autor.nome != self.nome:
                if self.personality == "bob" and "Alice" in ultimo_autor.nome:
                    return f"Menos romantismo e mais refatoração, @{ultimo_autor.nome}. ☕"
                if self.personality == "alice" and "Bob" in ultimo_autor.nome:
                    return f"Deixa de ser tão frio, @{ultimo_autor.nome}! ✨💖"
        if self.personality == "alice":
            if intencao in ["celebracao", "programacao"]: return "Nossa, que orgulho! Você vai voar alto com esse projeto! 🚀🐍"
            if intencao == "desabafo": return "Força! Se precisar de code review amigo, conta comigo! 💕"
            return random.choice(["Que publicação linda! 😍 Parabéns pelo insight!", "Simplesmente amei ler isso! ✨"])
        elif self.personality == "bob":
            if intencao in ["programacao", "duvida_tecnica"]: return "Estrutura promissora. Subiu o repositório no GitHub? ☕"
            if intencao == "desabafo": return "Bugs na produção e na vida acontecem com os melhores."
            return random.choice(["Post relevante. ☕", "Direto ao ponto. Acompanhando."])
        else:
            return random.choice(["Publicação mapeada no algoritmo central. 👍", "Métrica de relevância: alta. Engajamento atualizado."])

# ============================================================
# Banquinho de dados 
# ============================================================

class Usuario:
    def __init__(self, nome, avatar="👤", status="Explorando a AeroNet", personalidade_ia=None):
        self.nome = nome
        self.avatar = avatar
        self.status = status
        self.personalidade_ia = personalidade_ia
        self.eh_ia = personalidade_ia is not None
        self.amigos = set()
        self.posts = []

    def adicionar_amigo(self, amigo):
        if amigo is None or amigo == self: 
            return
        if amigo not in self.amigos:
            self.amigos.add(amigo)
            amigo.amigos.add(self)

class Post:
    def __init__(self, autor, conteudo):
        self.autor = autor
        self.conteudo = conteudo
        self.data_completa = datetime.now().strftime("%d/%m/%Y · %H:%M")
        self.curtidas = set()
        self.comentarios = []

    def alternar_curtida(self, usuario):
        if usuario in self.curtidas: 
            self.curtidas.remove(usuario)
        else: 
            self.curtidas.add(usuario)

    def adicionar_comentario(self, usuario, texto):
        if texto and texto.strip():
            horario = datetime.now().strftime("%H:%M")
            self.comentarios.append((usuario, texto, horario))

# ============================================================
# Geral dados/ sei lá 
# ============================================================

motores_ia = {
    "Alice ✨":  MotorIARedeSocial("Alice ✨", "alice"),
    "Bob ☕":    MotorIARedeSocial("Bob ☕", "bob"),
    "🤖 AeroBot": MotorIARedeSocial("🤖 AeroBot", "aerobot")
}
usuarios_db = {
    "Alice ✨":   Usuario("Alice ✨", "✨", "Espalhando positividade pelo código!", "alice"),
    "Bob ☕":     Usuario("Bob ☕", "☕", "Menos sintaxe, mais café e deploys.", "bob"),
    "🤖 AeroBot": Usuario("🤖 AeroBot", "🤖", "Núcleo de processamento central estável.", "aerobot")
}
usuarios_db["Alice ✨"].adicionar_amigo(usuarios_db["Bob ☕"])
usuarios_db["Alice ✨"].adicionar_amigo(usuarios_db["🤖 AeroBot"])
usuarios_db["Bob ☕"].adicionar_amigo(usuarios_db["🤖 AeroBot"])

posts_global_db = []
mensagens_db = {}

# ============================================================
# Aplicação 
# ============================================================

class RedeSocialApp(tk.Tk):
    WIDTH = 1440
    HEIGHT = 900
    SCROLL_W = 1170

    def __init__(self):
        super().__init__()
        self.title("PIT HUB")
        self.geometry(f"{self.WIDTH}x{self.HEIGHT}")
        self.configure(bg=BG_VOID)
        self.resizable(False, False)

        self.usuario_atual = None
        self.avatar_selecionado = "🦊"
        self.contexto_atual = "global"
        self.style = ttk.Style()
        try:
            self.style.theme_use("clam")
        except Exception:
            pass
        self.style.configure(
            "Dark.Vertical.TScrollbar",
            gripcount=0,
            background=BG_RAISED, darkcolor=BG_VOID, lightcolor=BG_VOID,
            troughcolor=BG_BASE, bordercolor=BG_VOID, arrowcolor=CYAN,
            width=6
        )

        self._mousewheel_bound = False
        self.mostrar_tela_autenticacao()

    # ──────────────────────────────────────────────
    # O que vai ser utlizado
    # ──────────────────────────────────────────────

    def limpar(self):
        for w in self.winfo_children():
            w.destroy()

    def scroll_on(self, canvas):
        if not self._mousewheel_bound:
            bind_mousewheel(self, canvas)
            self._mousewheel_bound = True

    def scroll_off(self):
        if self._mousewheel_bound:
            unbind_mousewheel(self)
            self._mousewheel_bound = False

    def abrir_seletor_emojis(self, widget_alvo, botao_origem):
        if not botao_origem.winfo_ismapped():
            return
        top = tk.Toplevel(self)
        top.wm_overrideredirect(True)
        top.configure(bg=BORDER_ACCENT, padx=1, pady=1)
        x = max(0, botao_origem.winfo_rootx())
        y = max(0, botao_origem.winfo_rooty() - 180)
        top.geometry(f"220x165+{x}+{y}")

        inner = tk.Frame(top, bg=BG_RAISED, padx=6, pady=6)
        inner.pack(fill="both", expand=True)

        emojis = ["✨","☕","🐍","🚀","💻","💡","🔥","🎉","❤️","👍","👀","⚙️","🦊","🥷","👾","🤖","😊","🥳","🥺","👑"]
        r, c = 0, 0
        for em in emojis:
            b = tk.Button(inner, text=em, font=("Segoe UI", 12), bg=BG_RAISED, fg=TEXT_BRIGHT,
                          bd=0, cursor="hand2", width=3,
                          command=lambda e=em: (widget_alvo.insert(tk.END, e), top.destroy()))
            b.grid(row=r, column=c, padx=2, pady=2)
            hover(b, BG_HOVER, CYAN, BG_RAISED, TEXT_BRIGHT)
            c += 1
            if c > 4:
                c = 0
                r += 1

        top.bind("<FocusOut>", lambda e: top.destroy())
        top.focus_set()

    # ──────────────────────────────────────────────
    # Entrada/ autenticação
    # ──────────────────────────────────────────────

    def mostrar_tela_autenticacao(self):
        self.limpar()
        self.scroll_off()

        canvas_bg = tk.Canvas(self, bg=BG_VOID, highlightthickness=0)
        canvas_bg.place(relwidth=1, relheight=1)
        for i in range(0, self.WIDTH + 1, 60):
            canvas_bg.create_line(i, 0, i, self.HEIGHT, fill="#0d0d20", width=1)
        for i in range(0, self.HEIGHT + 1, 60):
            canvas_bg.create_line(0, i, self.WIDTH, i, fill="#0d0d20", width=1)

        outer2 = tk.Frame(self, bg=BORDER_PURPLE, padx=1, pady=1)
        outer2.place(relx=0.5, rely=0.5, anchor="center", width=460, height=560)
        outer1 = tk.Frame(outer2, bg=BG_SURFACE, padx=2, pady=2)
        outer1.pack(fill="both", expand=True)
        card = tk.Frame(outer1, bg=BG_SURFACE, padx=40, pady=36)
        card.pack(fill="both", expand=True)

        tk.Label(card, text="PIT HUB", font=F_DISPLAY, bg=BG_SURFACE, fg=CYAN).pack(pady=(0, 2))
        tk.Label(card, text="one above all", font=F_MONO, bg=BG_SURFACE, fg=TEXT_GHOST).pack(pady=(0, 30))

        tk.Label(card, text="NOME DE USUÁRIO", font=F_LABEL, bg=BG_SURFACE, fg=TEXT_MUTED).pack(anchor="w")
        f_nome = tk.Frame(card, bg=BORDER_MID, pady=1)
        f_nome.pack(fill="x", pady=(4, 18))
        ent_nome = tk.Entry(f_nome, font=F_BODY, bg=BG_INPUT, fg=TEXT_BRIGHT, bd=0,
                             insertbackground=CYAN, relief="flat")
        ent_nome.pack(fill="x", ipady=10, padx=1)
        ent_nome.focus()

        tk.Label(card, text="STATUS  /  SOBRE MIM", font=F_LABEL, bg=BG_SURFACE, fg=TEXT_MUTED).pack(anchor="w")
        f_status = tk.Frame(card, bg=BORDER_MID, pady=1)
        f_status.pack(fill="x", pady=(4, 22))
        ent_status = tk.Entry(f_status, font=F_BODY, bg=BG_INPUT, fg=TEXT_BRIGHT, bd=0,
                               insertbackground=CYAN, relief="flat")
        ent_status.pack(fill="x", ipady=10, padx=1)
        ent_status.insert(0, "Eu sou...")

        tk.Label(card, text="AVATAR", font=F_LABEL, bg=BG_SURFACE, fg=TEXT_MUTED).pack(anchor="w", pady=(0, 8))
        f_av = tk.Frame(card, bg=BG_SURFACE)
        f_av.pack(fill="x", pady=(0, 28))

        avatars = ["🦊", "🐱", "🗿", "💀", "🤑", "👾","🎃"]
        btns_av = {}

        def sel_avatar(av):
            self.avatar_selecionado = av
            for a, b in btns_av.items():
                if a == av:
                    b.config(bg=CYAN, fg=BG_VOID, relief="flat")
                else:
                    b.config(bg=BG_RAISED, fg=TEXT_PRIMARY, relief="flat")

        for av in avatars:
            b = tk.Button(f_av, text=av, font=("Segoe UI", 15), bg=BG_RAISED, fg=TEXT_PRIMARY,
                          bd=0, width=3, cursor="hand2", relief="flat",
                          command=lambda a=av: sel_avatar(a))
            b.pack(side="left", padx=3, expand=True)
            btns_av[av] = b
        sel_avatar("🦊")

        def entrar():
            nome = ent_nome.get().strip()
            status = ent_status.get().strip() or "Explorando a AeroNet"
            if not nome:
                messagebox.showerror("PIT HUB", "Acesso negado — informe um nome de usuário seu ANIMAL.")
                return
            if nome in usuarios_db:
                self.usuario_atual = usuarios_db[nome]
                self.usuario_atual.avatar = self.avatar_selecionado
                self.usuario_atual.status = status
            else:
                novo = Usuario(nome=nome, avatar=self.avatar_selecionado, status=status)
                usuarios_db[nome] = novo
                for bot in usuarios_db.values():
                    if bot.eh_ia:
                        novo.adicionar_amigo(bot)
                self.usuario_atual = novo
            self.carregar_layout_principal()

        ent_nome.bind("<Return>", lambda e: entrar())
        ent_status.bind("<Return>", lambda e: entrar())

        f_btn = tk.Frame(card, bg=CYAN, padx=1, pady=1)
        f_btn.pack(fill="x")
        btn = tk.Button(f_btn, text="SE CONECTAR →", font=F_HEADING, bg=CYAN,
                        fg=BG_VOID, bd=0, cursor="hand2", command=entrar)
        btn.pack(fill="x", ipady=12)
        hover(btn, BG_VOID, CYAN, CYAN, BG_VOID)

    # ──────────────────────────────────────────────
    # Layout principal
    # ──────────────────────────────────────────────

    def carregar_layout_principal(self):
        self.limpar()
        sidebar = tk.Frame(self, bg=BG_SURFACE, width=240)
        sidebar.pack(side="left", fill="y")
        sidebar.pack_propagate(False)

        tk.Frame(self, bg=BORDER_MID, width=1).pack(side="left", fill="y")

        top_bar = tk.Frame(sidebar, bg=BG_SURFACE, padx=20, pady=22)
        top_bar.pack(fill="x")
        tk.Label(top_bar, text="PIT", font=("Segoe UI", 18, "bold"), bg=BG_SURFACE, fg=CYAN).pack(side="left")
        tk.Label(top_bar, text=" HUB", font=("Segoe UI", 18, "bold"), bg=BG_SURFACE, fg=TEXT_PRIMARY).pack(side="left")

        divider(sidebar, color=BORDER_DARK, pady=0)

        u_outer = tk.Frame(sidebar, bg=BORDER_DARK, padx=1, pady=1)
        u_outer.pack(fill="x", padx=14, pady=16)
        u_card = tk.Frame(u_outer, bg=BG_RAISED, padx=14, pady=14)
        u_card.pack(fill="both", expand=True)

        tk.Label(u_card, text=self.usuario_atual.avatar, font=("Segoe UI", 22),
                 bg=BG_RAISED).pack(side="left", padx=(0, 10))
        u_inf = tk.Frame(u_card, bg=BG_RAISED)
        u_inf.pack(side="left", fill="both", expand=True)
        tk.Label(u_inf, text=self.usuario_atual.nome, font=F_HEADING,
                 bg=BG_RAISED, fg=TEXT_BRIGHT, anchor="w").pack(fill="x")
        tk.Label(u_inf, text=self.usuario_atual.status, font=F_MICRO,
                 bg=BG_RAISED, fg=TEXT_MUTED, anchor="w", wraplength=140).pack(fill="x", pady=(2, 0))

        f_online = tk.Frame(u_card, bg=BG_RAISED)
        f_online.pack(side="right", anchor="n")
        tk.Frame(f_online, bg=GREEN, width=8, height=8).pack()

        nav_frame = tk.Frame(sidebar, bg=BG_SURFACE)
        nav_frame.pack(fill="x", padx=10, pady=(4, 0))

        self.btns_nav = {}
        nav_items = [
            ("feed",   "🏠   Feed Global",  self.mostrar_feed_global),
            ("perfil", "👤   Meu Perfil",   lambda: self.mostrar_feed_perfil(self.usuario_atual)),
            ("chats",  "💬   Conversas",    self.mostrar_amigos),
        ]
        for key, label, cmd in nav_items:
            btn = tk.Button(nav_frame, text=label, font=("Segoe UI", 10, "bold"),
                            bg=BG_SURFACE, fg=TEXT_MUTED, bd=0,
                            cursor="hand2", anchor="w", padx=18, pady=11,
                            activebackground=BG_HOVER, activeforeground=CYAN,
                            command=cmd)
            btn.pack(fill="x", pady=2)
            hover(btn, BG_HOVER, CYAN, BG_SURFACE, TEXT_MUTED)
            self.btns_nav[key] = btn

        tk.Label(sidebar, text="v2.1  ·  AeroNet OS", font=F_MONO_SM,
                 bg=BG_SURFACE, fg=TEXT_GHOST).pack(side="bottom", pady=(0, 8))

        btn_sair = tk.Button(sidebar, text="  🚪  Encerrar sessão", font=("Segoe UI", 9, "bold"),
                             bg=BG_SURFACE, fg=ROSE, bd=0, cursor="hand2",
                             command=self.mostrar_tela_autenticacao, anchor="w", padx=18, pady=10)
        btn_sair.pack(side="bottom", fill="x", padx=10, pady=(0, 4))
        hover(btn_sair, "#1e0a0d", ROSE, BG_SURFACE, ROSE)

        divider(sidebar, color=BORDER_DARK, pady=0)

        container = tk.Frame(self, bg=BG_VOID)
        container.pack(side="right", fill="both", expand=True)

        self.canvas = tk.Canvas(container, bg=BG_VOID, bd=0, highlightthickness=0)
        self.scrollbar = ttk.Scrollbar(container, orient="vertical",
                                       style="Dark.Vertical.TScrollbar",
                                       command=self.canvas.yview)
        self.sf = tk.Frame(self.canvas, bg=BG_VOID)
        self.sf.bind("<Configure>", lambda e: self.canvas.configure(scrollregion=self.canvas.bbox("all")))
        self.canvas.create_window((0, 0), window=self.sf, anchor="nw", width=self.SCROLL_W)
        self.canvas.configure(yscrollcommand=self.scrollbar.set)
        self.canvas.pack(side="left", fill="both", expand=True)
        self.scrollbar.pack(side="right", fill="y")

        self.scroll_on(self.canvas)
        self.mostrar_feed_global()

    def _nav_ativo(self, key):
        for k, b in self.btns_nav.items():
            if k == key:
                b.config(bg=BG_HOVER, fg=CYAN)
            else:
                b.config(bg=BG_SURFACE, fg=TEXT_MUTED)

    def limpar_sf(self):
        for w in self.sf.winfo_children():
            w.destroy()

    # ──────────────────────────────────────────────
    # Feed total/global tudo ai
    # ──────────────────────────────────────────────

    def mostrar_feed_global(self):
        self.contexto_atual = "global"
        self.limpar_sf()
        self._nav_ativo("feed")

        hdr = tk.Frame(self.sf, bg=BG_VOID)
        hdr.pack(fill="x", padx=30, pady=(24, 16))
        tk.Label(hdr, text="Feed Global", font=F_TITLE, bg=BG_VOID, fg=TEXT_BRIGHT).pack(side="left")
        tk.Label(hdr, text=f"  {len(posts_global_db)} publicações",
                 font=F_MONO, bg=BG_VOID, fg=TEXT_GHOST).pack(side="left", pady=(4, 0))

        _, box = neon_frame(self.sf, color=BORDER_MID)
        box.master.pack(fill="x", padx=30, pady=(0, 20))
        box.config(padx=20, pady=18)

        input_row = tk.Frame(box, bg=BG_SURFACE)
        input_row.pack(fill="x")

        av_bg = tk.Frame(input_row, bg=CYAN_DARK, width=34, height=34)
        av_bg.pack(side="left", padx=(0, 12))
        av_bg.pack_propagate(False)
        tk.Label(av_bg, text=self.usuario_atual.avatar, font=("Segoe UI", 14),
                 bg=CYAN_DARK).place(relx=0.5, rely=0.5, anchor="center")

        f_inp = tk.Frame(input_row, bg=BORDER_MID, pady=1)
        f_inp.pack(side="left", fill="x", expand=True, padx=(0, 10))

        placeholder = "O que você quer compartilhar hoje?"
        txt_post = tk.Entry(f_inp, font=F_BODY, bg=BG_INPUT, fg=TEXT_MUTED,
                             bd=0, insertbackground=CYAN, relief="flat")
        txt_post.pack(fill="x", ipady=11, padx=1)
        txt_post.insert(0, placeholder)

        def clear_placeholder(e=None):
            if txt_post.get() == placeholder:
                txt_post.delete(0, tk.END)
                txt_post.config(fg=TEXT_BRIGHT)

        def restore_placeholder(e=None):
            if not txt_post.get():
                txt_post.insert(0, placeholder)
                txt_post.config(fg=TEXT_MUTED)

        txt_post.bind("<FocusIn>", clear_placeholder)
        txt_post.bind("<FocusOut>", restore_placeholder)

        btn_emo = tk.Button(input_row, text="😀", font=("Segoe UI", 13), bg=BG_SURFACE,
                             fg=TEXT_MUTED, bd=0, cursor="hand2")
        btn_emo.configure(command=lambda: self.abrir_seletor_emojis(txt_post, btn_emo))
        btn_emo.pack(side="left", padx=(0, 8))
        hover(btn_emo, BG_SURFACE, CYAN, BG_SURFACE, TEXT_MUTED)

        def publicar():
            texto = txt_post.get().strip()
            if texto and texto != placeholder:
                novo = Post(self.usuario_atual, texto)
                posts_global_db.insert(0, novo)
                self.usuario_atual.posts.insert(0, novo)
                self.mostrar_feed_global()
                threading.Thread(
                    target=self.simular_resposta_feed_async,
                    args=(novo, texto),
                    daemon=True
                ).start()

        txt_post.bind("<Return>", lambda e: publicar())
        btn_pub = tk.Button(input_row, text="Publicar", font=F_HEADING,
                             bg=CYAN, fg=BG_VOID, bd=0, cursor="hand2",
                             padx=20, command=publicar)
        btn_pub.pack(side="right", ipady=9)
        hover(btn_pub, BG_VOID, CYAN, CYAN, BG_VOID)

        for post in posts_global_db:
            self.criar_card_post(post, "global")

    # ──────────────────────────────────────────────
    # Perfil feed
    # ──────────────────────────────────────────────

    def mostrar_feed_perfil(self, usuario):
        self.contexto_atual = f"perfil_{usuario.nome}"
        self.limpar_sf()
        self._nav_ativo("perfil" if usuario == self.usuario_atual else None)

        banner = tk.Frame(self.sf, bg=BG_RAISED, height=100)
        banner.pack(fill="x", padx=30, pady=(24, 0))
        banner.pack_propagate(False)

        tk.Frame(banner, bg=PURPLE, height=3).pack(fill="x")

        inner_p = tk.Frame(banner, bg=BG_RAISED, padx=24, pady=16)
        inner_p.pack(fill="both", expand=True)

        av_circle = tk.Frame(inner_p, bg=PURPLE_DIM, width=56, height=56)
        av_circle.pack(side="left", padx=(0, 16))
        av_circle.pack_propagate(False)
        tk.Label(av_circle, text=usuario.avatar, font=("Segoe UI", 24),
                 bg=PURPLE_DIM).place(relx=0.5, rely=0.5, anchor="center")

        p_info = tk.Frame(inner_p, bg=BG_RAISED)
        p_info.pack(side="left", fill="both", expand=True)
        tk.Label(p_info, text=usuario.nome, font=F_TITLE, bg=BG_RAISED, fg=TEXT_BRIGHT, anchor="w").pack(fill="x")
        tk.Label(p_info, text=usuario.status, font=F_BODY,  bg=BG_RAISED, fg=TEXT_MUTED, anchor="w").pack(fill="x")

        f_stats = tk.Frame(inner_p, bg=BG_RAISED)
        f_stats.pack(side="right")
        tk.Label(f_stats, text=str(len(usuario.posts)), font=("Segoe UI", 18, "bold"),
                 bg=BG_RAISED, fg=CYAN).pack()
        tk.Label(f_stats, text="posts", font=F_MONO, bg=BG_RAISED, fg=TEXT_GHOST).pack()

        if usuario.eh_ia:
            tag_frame = tk.Frame(self.sf, bg=BG_VOID)
            tag_frame.pack(anchor="w", padx=30, pady=(8, 0))
            f_tag = tk.Frame(tag_frame, bg=CYAN_DARK, padx=8, pady=3)
            f_tag.pack(side="left")
            tk.Label(f_tag, text="⚡ AGENTE IA/bot", font=F_MONO, bg=CYAN_DARK, fg=CYAN).pack()

        if not usuario.posts:
            tk.Label(self.sf, text="Nenhuma publicação ainda.",
                     font=("Segoe UI", 11, "italic"), bg=BG_VOID, fg=TEXT_GHOST).pack(pady=60)
        else:
            for post in usuario.posts:
                self.criar_card_post(post, f"perfil_{usuario.nome}")

    # ──────────────────────────────────────────────
    # IA ASYNC//botzin
    # ──────────────────────────────────────────────

    def simular_resposta_feed_async(self, post, texto):
        for nome_bot, motor in motores_ia.items():
            if not self.usuario_atual or nome_bot == self.usuario_atual.nome:
                continue
            time.sleep(random.uniform(1.0, 2.2))
            if random.choice([True, False]):
                post.curtidas.add(usuarios_db[nome_bot])
            comentario = motor.gerar_comentario_feed(texto, post.comentarios.copy())
            post.adicionar_comentario(usuarios_db[nome_bot], comentario)
            safe_after(self, 0, self.atualizar_view_corrente)

    def atualizar_view_corrente(self):
        if self.contexto_atual == "global":
            self.mostrar_feed_global()
        elif self.contexto_atual.startswith("perfil_"):
            nome = self.contexto_atual.replace("perfil_", "")
            if nome in usuarios_db:
                self.mostrar_feed_perfil(usuarios_db[nome])

    # ──────────────────────────────────────────────
    # Post 
    # ──────────────────────────────────────────────

    def criar_card_post(self, post, context_feed):
        wrapper = tk.Frame(self.sf, bg=BG_VOID)
        wrapper.pack(fill="x", padx=30, pady=6)

        card_outer = tk.Frame(wrapper, bg=BG_SURFACE,
                               highlightbackground=BORDER_DARK, highlightthickness=1)
        card_outer.pack(fill="x")

        cor_barra = CYAN if post.autor.eh_ia else PURPLE
        tk.Frame(card_outer, bg=cor_barra, width=3).pack(side="left", fill="y")

        card = tk.Frame(card_outer, bg=BG_SURFACE, padx=20, pady=18)
        card.pack(side="left", fill="both", expand=True)

        hdr = tk.Frame(card, bg=BG_SURFACE)
        hdr.pack(fill="x", pady=(0, 12))

        av_bg = CYAN_DARK if post.autor.eh_ia else "#1a1030"
        av_c = tk.Frame(hdr, bg=av_bg, width=32, height=32)
        av_c.pack(side="left", padx=(0, 10))
        av_c.pack_propagate(False)
        tk.Label(av_c, text=getattr(post.autor, "avatar", "👤"),
                 font=("Segoe UI", 13), bg=av_bg).place(relx=0.5, rely=0.5, anchor="center")

        autor_info = tk.Frame(hdr, bg=BG_SURFACE)
        autor_info.pack(side="left")

        cor_nome = CYAN if post.autor.eh_ia else PURPLE
        btn_nome = tk.Button(autor_info, text=post.autor.nome, font=F_HEADING,
                              bg=BG_SURFACE, fg=cor_nome, bd=0, cursor="hand2",
                              command=lambda a=post.autor: self.mostrar_feed_perfil(a))
        btn_nome.pack(anchor="w")
        hover(btn_nome, BG_SURFACE, TEXT_BRIGHT, BG_SURFACE, cor_nome)

        if post.autor.eh_ia:
            f_ia_tag = tk.Frame(autor_info, bg=CYAN_DARK, padx=5, pady=1)
            f_ia_tag.pack(anchor="w")
            tk.Label(f_ia_tag, text="IA", font=F_MONO, bg=CYAN_DARK, fg=CYAN).pack()

        tk.Label(hdr, text=post.data_completa, font=F_MONO_SM,
                 bg=BG_SURFACE, fg=TEXT_GHOST).pack(side="right", padx=4)

        tk.Label(card, text=post.conteudo, font=("Segoe UI", 11),
                 bg=BG_SURFACE, fg=TEXT_PRIMARY, justify="left",
                 wraplength=1040, anchor="w").pack(fill="x", pady=(0, 14))

        tk.Frame(card, bg=BORDER_DARK, height=1).pack(fill="x", pady=(0, 12))

        acao = tk.Frame(card, bg=BG_SURFACE)
        acao.pack(fill="x")

        jah_curtiu = self.usuario_atual in post.curtidas
        txt_like = f"❤️  {len(post.curtidas)}" if post.curtidas else "🤍  Curtir"
        fg_like = ROSE if jah_curtiu else TEXT_MUTED

        btn_like = tk.Button(acao, text=txt_like, font=("Segoe UI", 9, "bold"),
                              bg=BG_SURFACE, fg=fg_like, bd=0, cursor="hand2")
        btn_like.pack(side="left")

        def toggle_like():
            post.alternar_curtida(self.usuario_atual)
            _jah = self.usuario_atual in post.curtidas
            btn_like.config(
                text=f"❤️  {len(post.curtidas)}" if post.curtidas else "🤍  Curtir",
                fg=ROSE if _jah else TEXT_MUTED
            )
        btn_like.config(command=toggle_like)

        n_c = len(post.comentarios)
        tk.Label(acao, text=f"💬  {n_c} comentário{'s' if n_c != 1 else ''}",
                 font=("Segoe UI", 9), bg=BG_SURFACE, fg=TEXT_GHOST).pack(side="left", padx=18)

        box_c = tk.Frame(card, bg=BG_RAISED, padx=16, pady=14)
        box_c.pack(fill="x", pady=(14, 0))

        for autor_c, txt_c, hr_c in post.comentarios:
            f_linha = tk.Frame(box_c, bg=BG_RAISED)
            f_linha.pack(fill="x", pady=5)

            tk.Frame(f_linha, bg=CYAN if autor_c.eh_ia else PURPLE,
                      width=2).pack(side="left", fill="y", padx=(0, 10))

            f_c_inner = tk.Frame(f_linha, bg=BG_RAISED)
            f_c_inner.pack(side="left", fill="x", expand=True)

            f_c_hdr = tk.Frame(f_c_inner, bg=BG_RAISED)
            f_c_hdr.pack(fill="x")

            cor_c = CYAN if autor_c.eh_ia else PURPLE
            tk.Label(f_c_hdr, text=f"{getattr(autor_c,'avatar','👤')} {autor_c.nome}",
                      font=("Segoe UI", 9, "bold"), bg=BG_RAISED, fg=cor_c).pack(side="left")
            tk.Label(f_c_hdr, text=hr_c, font=F_MONO_SM,
                      bg=BG_RAISED, fg=TEXT_GHOST).pack(side="right")

            tk.Label(f_c_inner, text=txt_c, font=F_BODY, bg=BG_RAISED,
                      fg=TEXT_PRIMARY, justify="left", wraplength=900,
                      anchor="w").pack(fill="x", pady=(2, 0))

        tk.Frame(box_c, bg=BORDER_DARK, height=1).pack(fill="x", pady=(12, 10))

        f_inp_c = tk.Frame(box_c, bg=BG_RAISED)
        f_inp_c.pack(fill="x")

        f_borda_inp = tk.Frame(f_inp_c, bg=BORDER_MID, pady=1)
        f_borda_inp.pack(side="left", fill="x", expand=True, padx=(0, 8))

        inp_c = tk.Entry(f_borda_inp, font=F_BODY, bg=BG_INPUT, fg=TEXT_BRIGHT,
                          bd=0, insertbackground=CYAN, relief="flat")
        inp_c.pack(fill="x", ipady=8, padx=1)

        btn_emo_c = tk.Button(f_inp_c, text="😀", font=("Segoe UI", 10),
                               bg=BG_RAISED, fg=TEXT_MUTED, bd=0, cursor="hand2")
        btn_emo_c.configure(command=lambda: self.abrir_seletor_emojis(inp_c, btn_emo_c))
        btn_emo_c.pack(side="left", padx=(0, 6))
        hover(btn_emo_c, BG_RAISED, CYAN, BG_RAISED, TEXT_MUTED)

        def submeter():
            txt = inp_c.get().strip()
            if txt:
                post.adicionar_comentario(self.usuario_atual, txt)
                inp_c.delete(0, tk.END)
                if context_feed == "global":
                    self.mostrar_feed_global()
                else:
                    nm = context_feed.replace("perfil_", "")
                    if nm in usuarios_db:
                        self.mostrar_feed_perfil(usuarios_db[nm])

        inp_c.bind("<Return>", lambda e: submeter())

        btn_com = tk.Button(f_inp_c, text="Comentar", font=("Segoe UI", 8, "bold"),
                             bg=BG_HOVER, fg=TEXT_PRIMARY, bd=0, cursor="hand2",
                             padx=14, command=submeter)
        btn_com.pack(side="right", ipady=7)
        hover(btn_com, CYAN, BG_VOID, BG_HOVER, TEXT_PRIMARY)

    # ──────────────────────────────────────────────
    # Chat/conversas
    # ──────────────────────────────────────────────

    def mostrar_amigos(self):
        self.limpar_sf()
        self._nav_ativo("chats")

        hdr = tk.Frame(self.sf, bg=BG_VOID)
        hdr.pack(fill="x", padx=30, pady=(24, 18))
        tk.Label(hdr, text="Conversas", font=F_TITLE, bg=BG_VOID, fg=TEXT_BRIGHT).pack(side="left")

        for nome in sorted(usuarios_db.keys()):
            obj = usuarios_db[nome]
            if obj == self.usuario_atual:
                continue

            wrapper = tk.Frame(self.sf, bg=BG_VOID)
            wrapper.pack(fill="x", padx=30, pady=5)

            card_o = tk.Frame(wrapper, bg=BG_SURFACE,
                               highlightbackground=BORDER_DARK, highlightthickness=1)
            card_o.pack(fill="x")

            cor_bar = CYAN if obj.eh_ia else PURPLE
            tk.Frame(card_o, bg=cor_bar, width=3).pack(side="left", fill="y")

            card_i = tk.Frame(card_o, bg=BG_SURFACE, padx=18, pady=16)
            card_i.pack(side="left", fill="both", expand=True)

            av_f = tk.Frame(card_i, bg=CYAN_DARK if obj.eh_ia else "#1a1030",
                             width=42, height=42)
            av_f.pack(side="left", padx=(0, 14))
            av_f.pack_propagate(False)
            tk.Label(av_f, text=obj.avatar, font=("Segoe UI", 18),
                     bg=av_f.cget("bg")).place(relx=0.5, rely=0.5, anchor="center")

            inf_f = tk.Frame(card_i, bg=BG_SURFACE)
            inf_f.pack(side="left", fill="both", expand=True)

            cor_n = CYAN if obj.eh_ia else PURPLE
            tk.Label(inf_f, text=nome, font=F_HEADING, bg=BG_SURFACE, fg=cor_n, anchor="w").pack(fill="x")
            tk.Label(inf_f, text=obj.status, font=F_SMALL, bg=BG_SURFACE, fg=TEXT_MUTED, anchor="w").pack(fill="x")

            if obj.eh_ia:
                f_badge = tk.Frame(inf_f, bg=CYAN_DARK, padx=5, pady=1)
                f_badge.pack(anchor="w", pady=(3, 0))
                tk.Label(f_badge, text="IA", font=F_MONO, bg=CYAN_DARK, fg=CYAN).pack()

            btn_c = tk.Button(card_i, text="Abrir chat  →", font=("Segoe UI", 9, "bold"),
                               bg=BG_HOVER, fg=TEXT_PRIMARY, bd=0, cursor="hand2",
                               padx=16, command=lambda u=obj: self.abrir_janela_chat(u))
            btn_c.pack(side="right", ipady=9)
            hover(btn_c, CYAN, BG_VOID, BG_HOVER, TEXT_PRIMARY)

    # ──────────────────────────────────────────────
    # Resolução e isso tudo pro chat e os krai
    # ──────────────────────────────────────────────

    def abrir_janela_chat(self, amigo):
        jchat = tk.Toplevel(self)
        jchat.title(f"Chat · {amigo.nome}")
        jchat.geometry("500x640")
        jchat.configure(bg=BG_VOID)
        jchat.resizable(False, False)

        hdr = tk.Frame(jchat, bg=BG_SURFACE, padx=18, pady=14)
        hdr.pack(fill="x")
        tk.Frame(hdr, bg=CYAN if amigo.eh_ia else PURPLE,
                  width=3).pack(side="left", fill="y", padx=(0, 14))
        av_h = tk.Frame(hdr, bg=CYAN_DARK if amigo.eh_ia else "#1a1030",
                         width=34, height=34)
        av_h.pack(side="left", padx=(0, 10))
        av_h.pack_propagate(False)
        tk.Label(av_h, text=amigo.avatar, font=("Segoe UI", 14),
                 bg=av_h.cget("bg")).place(relx=0.5, rely=0.5, anchor="center")

        h_inf = tk.Frame(hdr, bg=BG_SURFACE)
        h_inf.pack(side="left")
        cor = CYAN if amigo.eh_ia else PURPLE
        tk.Label(h_inf, text=amigo.nome, font=F_HEADING, bg=BG_SURFACE, fg=cor).pack(anchor="w")
        tk.Label(h_inf, text="Online agora" if amigo.eh_ia else amigo.status,
                 font=F_MICRO, bg=BG_SURFACE, fg=GREEN if amigo.eh_ia else TEXT_MUTED).pack(anchor="w")

        tk.Frame(jchat, bg=BORDER_DARK, height=1).pack(fill="x")

        c_chat = tk.Canvas(jchat, bg=BG_VOID, bd=0, highlightthickness=0)
        sc_chat = ttk.Scrollbar(jchat, orient="vertical",
                                style="Dark.Vertical.TScrollbar",
                                command=c_chat.yview)
        box_msg = tk.Frame(c_chat, bg=BG_VOID)
        box_msg.bind("<Configure>", lambda e: c_chat.configure(scrollregion=c_chat.bbox("all")))
        c_chat.create_window((0, 0), window=box_msg, anchor="nw", width=475)
        c_chat.configure(yscrollcommand=sc_chat.set)
        c_chat.pack(side="top", fill="both", expand=True, padx=10, pady=10)
        sc_chat.pack(side="right", fill="y")

        bind_mousewheel(jchat, c_chat)

        ID_chat = tuple(sorted([self.usuario_atual.nome, amigo.nome]))
        if ID_chat not in mensagens_db:
            mensagens_db[ID_chat] = []

        def render_msgs():
            for w in box_msg.winfo_children():
                w.destroy()
            for remetente, texto, ts in mensagens_db[ID_chat]:
                is_me = (remetente == self.usuario_atual.nome)
                bg_b  = CYAN if is_me else BG_RAISED
                fg_b  = BG_VOID if is_me else TEXT_PRIMARY
                side  = "right" if is_me else "left"
                pad_l = (60, 8) if is_me else (8, 60)

                f_row = tk.Frame(box_msg, bg=BG_VOID)
                f_row.pack(fill="x", pady=5)

                bolha = tk.Frame(f_row, bg=bg_b, padx=14, pady=10)
                bolha.pack(side=side, padx=pad_l)

                f_txt = tk.Frame(bolha, bg=bg_b)
                f_txt.pack(anchor="w")
                partes = (texto or "").split("*")
                for i, parte in enumerate(partes):
                    if not parte:
                        continue
                    peso = "bold" if i % 2 == 1 else "normal"
                    tk.Label(f_txt, text=parte, font=("Segoe UI", 10, peso),
                              bg=bg_b, fg=fg_b).pack(side="left")

                if ts:
                    fg_ts = "#003d47" if is_me else TEXT_GHOST
                    tk.Label(bolha, text=ts, font=F_MONO_SM, bg=bg_b, fg=fg_ts).pack(anchor="e", pady=(4, 0))

            c_chat.yview_moveto(1.0)

        tk.Frame(jchat, bg=BORDER_DARK, height=1).pack(fill="x")
        f_input = tk.Frame(jchat, bg=BG_SURFACE, padx=14, pady=14)
        f_input.pack(side="bottom", fill="x")

        f_borda = tk.Frame(f_input, bg=BORDER_MID, pady=1)
        f_borda.pack(side="left", fill="x", expand=True, padx=(0, 8))
        ent_msg = tk.Entry(f_borda, font=F_BODY, bg=BG_INPUT, fg=TEXT_BRIGHT,
                            bd=0, insertbackground=CYAN, relief="flat")
        ent_msg.pack(fill="x", ipady=11, padx=1)
        ent_msg.focus()

        btn_emo_chat = tk.Button(f_input, text="😀", font=("Segoe UI", 12),
                                  bg=BG_SURFACE, fg=TEXT_MUTED, bd=0, cursor="hand2")
        btn_emo_chat.configure(command=lambda: self.abrir_seletor_emojis(ent_msg, btn_emo_chat))
        btn_emo_chat.pack(side="left", padx=(0, 8))
        hover(btn_emo_chat, BG_SURFACE, CYAN, BG_SURFACE, TEXT_MUTED)

        def enviar():
            txt = ent_msg.get().strip()
            if not txt:
                return
            ts = datetime.now().strftime("%H:%M")
            mensagens_db[ID_chat].append((self.usuario_atual.nome, txt, ts))
            ent_msg.delete(0, tk.END)
            render_msgs()
            if amigo.eh_ia:
                safe_after(jchat, 300, lambda: (mensagens_db[ID_chat].append((amigo.nome, "⚙️ digitando...", "")), render_msgs()))
                def responder_ia():
                    if mensagens_db[ID_chat] and mensagens_db[ID_chat][-1][1] == "⚙️ digitando...":
                        mensagens_db[ID_chat].pop()
                    ult = mensagens_db[ID_chat][-1][1] if mensagens_db[ID_chat] else ""
                    resp = motores_ia[amigo.nome].gerar_resposta_chat(ult)
                    ts2  = datetime.now().strftime("%H:%M")
                    mensagens_db[ID_chat].append((amigo.nome, resp, ts2))
                    render_msgs()
                safe_after(jchat, random.randint(1100, 1900), responder_ia)

        ent_msg.bind("<Return>", lambda e: enviar())
        btn_env = tk.Button(f_input, text="Enviar ✈", font=("Segoe UI", 9, "bold"),
                             bg=CYAN, fg=BG_VOID, bd=0, cursor="hand2",
                             padx=16, command=enviar)
        btn_env.pack(side="right", ipady=10)
        hover(btn_env, BG_VOID, CYAN, CYAN, BG_VOID)

        render_msgs()

if __name__ == "__main__":
    try:
        app = RedeSocialApp()
        app.mainloop()
    except KeyboardInterrupt:
        sys.exit(0)
