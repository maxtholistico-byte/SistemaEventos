import java.io.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.*;

// Classe Usuário
class Usuario {
    private String nome;
    private String email;
    private String cidade;
    private List<Evento> eventosConfirmados = new ArrayList<>();

    public Usuario(String nome, String email, String cidade) {
        this.nome = nome;
        this.email = email;
        this.cidade = cidade;
    }

    public String getNome() {
        return nome;
    }

    public void confirmarEvento(Evento evento) {
        eventosConfirmados.add(evento);
    }

    public void cancelarEvento(Evento evento) {
        eventosConfirmados.remove(evento);
    }

    public List<Evento> getEventosConfirmados() {
        return eventosConfirmados;
    }
}

// Classe Evento
class Evento {
    private String nome;
    private String endereco;
    private String categoria;
    private LocalDateTime horario;
    private String descricao;

    public Evento(String nome, String endereco, String categoria, LocalDateTime horario, String descricao) {
        this.nome = nome;
        this.endereco = endereco;
        this.categoria = categoria;
        this.horario = horario;
        this.descricao = descricao;
    }

    public String getNome() { return nome; }
    public String getEndereco() { return endereco; }
    public String getCategoria() { return categoria; }
    public LocalDateTime getHorario() { return horario; }
    public String getDescricao() { return descricao; }

    public String toFileFormat() {
        return nome + ";" + endereco + ";" + categoria + ";" + horario + ";" + descricao;
    }

    public static Evento fromFileFormat(String line) {
        try {
            String[] parts = line.split(";");
            DateTimeFormatter fmt = DateTimeFormatter.ofPattern("yyyy-MM-dd'T'HH:mm");
            return new Evento(
                    parts[0],
                    parts[1],
                    parts[2],
                    LocalDateTime.parse(parts[3], fmt),
                    parts[4]
            );
        } catch (Exception e) {
            return null;
        }
    }

    @Override
    public String toString() {
        DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
        return "Evento: " + nome +
                " | Endereço: " + endereco +
                " | Categoria: " + categoria +
                " | Horário: " + horario.format(fmt) +
                " | Descrição: " + descricao;
    }
}

// Classe principal
public class SistemaEventos {
    private static final String FILE_NAME = "events.data";
    private static List<Evento> eventos = new ArrayList<>();
    private static Scanner sc = new Scanner(System.in);
    private static Usuario usuario;

    public static void main(String[] args) {
        carregarEventos();

        System.out.println("=== CADASTRO DO USUÁRIO ===");
        System.out.print("Nome: ");
        String nome = sc.nextLine();
        System.out.print("Email: ");
        String email = sc.nextLine();
        System.out.print("Cidade: ");
        String cidade = sc.nextLine();
        usuario = new Usuario(nome, email, cidade);

        int opcao;
        do {
            System.out.println("\n=== MENU ===");
            System.out.println("1 - Cadastrar evento");
            System.out.println("2 - Listar eventos");
            System.out.println("3 - Participar de evento");
            System.out.println("4 - Cancelar participação");
            System.out.println("5 - Meus eventos confirmados");
            System.out.println("6 - Salvar e sair");
            System.out.print("Escolha: ");
            opcao = Integer.parseInt(sc.nextLine());

            switch (opcao) {
                case 1 -> cadastrarEvento();
                case 2 -> listarEventos();
                case 3 -> participarEvento();
                case 4 -> cancelarParticipacao();
                case 5 -> listarEventosConfirmados();
                case 6 -> salvarEventos();
                default -> System.out.println("Opção inválida!");
            }
        } while (opcao != 6);
    }

    private static void cadastrarEvento() {
        System.out.println("\n=== CADASTRO DE EVENTO ===");
        System.out.print("Nome: ");
        String nome = sc.nextLine();
        System.out.print("Endereço: ");
        String endereco = sc.nextLine();
        System.out.print("Categoria (Festa, Show, Esporte, Cultural): ");
        String categoria = sc.nextLine();
        System.out.print("Horário (yyyy-MM-dd HH:mm): ");
        String horarioStr = sc.nextLine();
        LocalDateTime horario = LocalDateTime.parse(horarioStr.replace(" ", "T"));
        System.out.print("Descrição: ");
        String descricao = sc.nextLine();

        Evento evento = new Evento(nome, endereco, categoria, horario, descricao);
        eventos.add(evento);
        System.out.println("Evento cadastrado com sucesso!");
    }

    private static void listarEventos() {
        if (eventos.isEmpty()) {
            System.out.println("Nenhum evento cadastrado.");
            return;
        }

        eventos.sort(Comparator.comparing(Evento::getHorario));
        LocalDateTime agora = LocalDateTime.now();

        for (Evento e : eventos) {
            System.out.println(e);
            if (e.getHorario().isBefore(agora)) {
                System.out.println("   -> Este evento já ocorreu.");
            } else if (e.getHorario().isAfter(agora.minusMinutes(1)) && e.getHorario().isBefore(agora.plusHours(3))) {
                System.out.println("   -> Este evento está ocorrendo agora ou em breve!");
            }
        }
    }

    private static void participarEvento() {
        listarEventos();
        System.out.print("Digite o nome do evento que deseja participar: ");
        String nomeEvento = sc.nextLine();
        for (Evento e : eventos) {
            if (e.getNome().equalsIgnoreCase(nomeEvento)) {
                usuario.confirmarEvento(e);
                System.out.println("Participação confirmada!");
                return;
            }
        }
        System.out.println("Evento não encontrado.");
    }

    private static void cancelarParticipacao() {
        listarEventosConfirmados();
        System.out.print("Digite o nome do evento que deseja cancelar: ");
        String nomeEvento = sc.nextLine();
        for (Evento e : usuario.getEventosConfirmados()) {
            if (e.getNome().equalsIgnoreCase(nomeEvento)) {
                usuario.cancelarEvento(e);
                System.out.println("Participação cancelada!");
                return;
            }
        }
        System.out.println("Evento não encontrado.");
    }

    private static void listarEventosConfirmados() {
        List<Evento> confirmados = usuario.getEventosConfirmados();
        if (confirmados.isEmpty()) {
            System.out.println("Você não confirmou nenhum evento.");
        } else {
            confirmados.forEach(System.out::println);
        }
    }

    private static void salvarEventos() {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(FILE_NAME))) {
            for (Evento e : eventos) {
                writer.write(e.toFileFormat());
                writer.newLine();
            }
            System.out.println("Eventos salvos em " + FILE_NAME);
        } catch (IOException e) {
            System.out.println("Erro ao salvar eventos: " + e.getMessage());
        }
    }

    private static void carregarEventos() {
        File file = new File(FILE_NAME);
        if (!file.exists()) return;

        try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
            String line;
            while ((line = reader.readLine()) != null) {
                Evento e = Evento.fromFileFormat(line);
                if (e != null) eventos.add(e);
            }
        } catch (IOException e) {
            System.out.println("Erro ao carregar eventos: " + e.getMessage());
        }
    }
}
