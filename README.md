# sistema-de-cadastro

//+-------------------+           +-------------------+
//|      Usuario      |           |      Evento       |
//+-------------------+           +-------------------+
//| - Nome            |           | - Nome            |
//| - Email           |           | - Endereco        |
//| - Telefone        |           | - Categoria       |
//| - EventosConfirm. |*--------->| - Horario         |
//+-------------------+           | - Descricao       |
//| + Confirmar()     |           | - Participantes   |
//| + Cancelar()      |           +-------------------+
//+-------------------+

//+-------------------+
//| GerenciadorEventos|
//+-------------------+
//| - ListaEventos    |
//+-------------------+
//| + CadastrarEvento()|
//| + ListarEventos() |
//| + SalvarEventos() |
//| + CarregarEventos()|
//+-------------------+


using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;

namespace SistemaEventos
{
    // Enum (lista fixa) que define as categorias possíveis de eventos
    enum CategoriaEvento
    {
        Festa,
        Esporte,
        Show,
        Teatro,
        Palestra
    }

    // Classe que representa um Usuário do sistema
    class Usuario
    {
        // Atributos do usuário
        public string Nome { get; set; }
        public string Email { get; set; }
        public string Telefone { get; set; }

        // Lista de eventos que o usuário confirmou presença
        public List<Evento> EventosConfirmados { get; private set; } = new List<Evento>();

        // Método para confirmar presença em um evento
        public void ConfirmarPresenca(Evento evento)
        {
            if (!EventosConfirmados.Contains(evento))
            {
                EventosConfirmados.Add(evento); // Adiciona à lista do usuário
                evento.Participantes.Add(this); // Adiciona o usuário à lista de participantes do evento
                Console.WriteLine($"✅ Presença confirmada em: {evento.Nome}");
            }
        }

        // Método para cancelar presença em um evento
        public void CancelarPresenca(Evento evento)
        {
            if (EventosConfirmados.Contains(evento))
            {
                EventosConfirmados.Remove(evento); // Remove da lista do usuário
                evento.Participantes.Remove(this); // Remove o usuário da lista do evento
                Console.WriteLine($"❌ Presença cancelada em: {evento.Nome}");
            }
        }
    }

    // Classe que representa um Evento
    class Evento
    {
        // Atributos obrigatórios do evento
        public string Nome { get; set; }
        public string Endereco { get; set; }
        public CategoriaEvento Categoria { get; set; }
        public DateTime Horario { get; set; }
        public string Descricao { get; set; }

        // Lista de usuários que confirmaram presença neste evento
        public List<Usuario> Participantes { get; private set; } = new List<Usuario>();

        // Método que verifica se o evento está acontecendo agora
        public bool EstaOcorrendo()
        {
            return DateTime.Now >= Horario && DateTime.Now <= Horario.AddHours(2);
        }

        // Método que verifica se o evento já passou
        public bool JaPassou()
        {
            return DateTime.Now > Horario.AddHours(2);
        }
    }

    // Classe que gerencia todos os eventos do sistema
    class GerenciadorEventos
    {
        // Nome do arquivo onde os eventos serão salvos
        private const string ARQUIVO = "events.data";

        // Lista com todos os eventos cadastrados
        public List<Evento> ListaEventos { get; private set; } = new List<Evento>();

        // Método para cadastrar um novo evento
        public void CadastrarEvento(Evento ev)
        {
            ListaEventos.Add(ev);
            SalvarEventos(); // Sempre salva no arquivo após cadastrar
        }

        // Método para listar todos os eventos (ordenados pela data/hora)
        public void ListarEventos()
        {
            var eventosOrdenados = ListaEventos.OrderBy(e => e.Horario).ToList();
            foreach (var ev in eventosOrdenados)
            {
                // Define se o evento já passou, está ocorrendo ou é futuro
                string status = ev.EstaOcorrendo() ? "⏳ Em andamento" :
                                ev.JaPassou() ? "✅ Já ocorreu" : "📅 Futuro";

                Console.WriteLine($"{ev.Nome} - {ev.Categoria} - {ev.Horario} - {status}");
            }
        }

        // Método que salva os eventos no arquivo "events.data"
        public void SalvarEventos()
        {
            using (StreamWriter sw = new StreamWriter(ARQUIVO))
            {
                foreach (var ev in ListaEventos)
                {
                    // Cada evento é salvo em uma linha separada no formato texto
                    sw.WriteLine($"{ev.Nome};{ev.Endereco};{ev.Categoria};{ev.Horario};{ev.Descricao}");
                }
            }
        }

        // Método que carrega os eventos do arquivo quando o programa inicia
        public void CarregarEventos()
        {
            if (!File.Exists(ARQUIVO)) return; // Se o arquivo não existe, não faz nada

            string[] linhas = File.ReadAllLines(ARQUIVO);
            foreach (var linha in linhas)
            {
                var partes = linha.Split(';');
                Evento ev = new Evento
                {
                    Nome = partes[0],
                    Endereco = partes[1],
                    Categoria = Enum.Parse<CategoriaEvento>(partes[2]),
                    Horario = DateTime.Parse(partes[3]),
                    Descricao = partes[4]
                };
                ListaEventos.Add(ev);
            }
        }
    }

    // Classe principal que "roda" o sistema no console
    class Program
    {
        static void Main(string[] args)
        {
            // Cria o gerenciador e carrega os eventos salvos no arquivo
            GerenciadorEventos gerenciador = new GerenciadorEventos();
            gerenciador.CarregarEventos();

            Console.WriteLine("===== Sistema de Eventos =====");

            // Cadastro inicial do usuário
            Console.Write("Digite seu nome: ");
            string nome = Console.ReadLine();
            Console.Write("Digite seu e-mail: ");
            string email = Console.ReadLine();
            Console.Write("Digite seu telefone: ");
            string telefone = Console.ReadLine();

            Usuario usuario = new Usuario { Nome = nome, Email = email, Telefone = telefone };

            int opcao;
            do
            {
                // Exibe o menu de opções
                Console.WriteLine("\n--- MENU ---");
                Console.WriteLine("1 - Cadastrar Evento");
                Console.WriteLine("2 - Listar Eventos");
                Console.WriteLine("3 - Confirmar Presença");
                Console.WriteLine("4 - Cancelar Presença");
                Console.WriteLine("5 - Meus Eventos");
                Console.WriteLine("0 - Sair");
                Console.Write("Escolha: ");
                opcao = int.Parse(Console.ReadLine());

                switch (opcao)
                {
                    case 1:
                        // Cadastro de um novo evento
                        Evento ev = new Evento();
                        Console.Write("Nome: ");
                        ev.Nome = Console.ReadLine();
                        Console.Write("Endereço: ");
                        ev.Endereco = Console.ReadLine();
                        Console.Write("Categoria (Festa, Esporte, Show, Teatro, Palestra): ");
                        ev.Categoria = Enum.Parse<CategoriaEvento>(Console.ReadLine(), true);
                        Console.Write("Data e Hora (ex: 2025-09-08 20:00): ");
                        ev.Horario = DateTime.Parse(Console.ReadLine());
                        Console.Write("Descrição: ");
                        ev.Descricao = Console.ReadLine();

                        gerenciador.CadastrarEvento(ev);
                        break;

                    case 2:
                        // Mostra todos os eventos
                        gerenciador.ListarEventos();
                        break;

                    case 3:
                        // Confirmar presença em um evento
                        gerenciador.ListarEventos();
                        Console.Write("Digite o nome do evento: ");
                        string nomeEvento = Console.ReadLine();
                        var eventoConfirmar = gerenciador.ListaEventos.FirstOrDefault(e => e.Nome == nomeEvento);
                        if (eventoConfirmar != null) usuario.ConfirmarPresenca(eventoConfirmar);
                        else Console.WriteLine("Evento não encontrado!");
                        break;

                    case 4:
                        // Cancelar presença em um evento
                        Console.Write("Digite o nome do evento para cancelar: ");
                        string nomeCancelar = Console.ReadLine();
                        var eventoCancelar = usuario.EventosConfirmados.FirstOrDefault(e => e.Nome == nomeCancelar);
                        if (eventoCancelar != null) usuario.CancelarPresenca(eventoCancelar);
                        else Console.WriteLine("Você não confirmou presença nesse evento.");
                        break;

                    case 5:
                        // Mostrar todos os eventos que o usuário confirmou
                        Console.WriteLine("--- Seus eventos confirmados ---");
                        foreach (var e in usuario.EventosConfirmados)
                        {
                            Console.WriteLine($"{e.Nome} - {e.Horario}");
                        }
                        break;
                }

            } while (opcao != 0); // Repete até o usuário escolher sair
        }
    }
}
