# Lab-4-EDDA

**import java.util.*;

class Paciente implements Comparable<Paciente> {
    String nombre, apellido, id, estado, area;
    int categoria;
    long tiempoLlegada;
    Stack<String> historialCambios;

    public Paciente(String nombre, String apellido, String id, int categoria, long tiempoLlegada, String area) {
        this.nombre = nombre;
        this.apellido = apellido;
        this.id = id;
        this.categoria = categoria;
        this.tiempoLlegada = tiempoLlegada;
        this.estado = "en_espera";
        this.area = area;
        this.historialCambios = new Stack<>();
    }

    public long tiempoEsperaActual(long minutoActual) {
        return Math.max(0, minutoActual - tiempoLlegada);
    }

    public void registrarCambio(String descripcion) {
        historialCambios.push(descripcion);
    }

    public String obtenerUltimoCambio() {
        return historialCambios.isEmpty() ? null : historialCambios.peek();
    }

    public int compareTo(Paciente otro) {
        if (this.categoria != otro.categoria)
            return Integer.compare(this.categoria, otro.categoria);
        return Long.compare(this.tiempoLlegada, otro.tiempoLlegada);
    }
}

class AreaAtencion {
    String nombre;
    PriorityQueue<Paciente> pacientesHeap;
    int capacidadMaxima;

    public AreaAtencion(String nombre, int capacidadMaxima) {
        this.nombre = nombre;
        this.capacidadMaxima = capacidadMaxima;
        this.pacientesHeap = new PriorityQueue<>();
    }

    public void ingresarPaciente(Paciente p) {
        if (!estaSaturada()) pacientesHeap.offer(p);
    }

    public Paciente atenderPaciente() {
        return pacientesHeap.poll();
    }

    public boolean estaSaturada() {
        return pacientesHeap.size() >= capacidadMaxima;
    }
}

class Hospital {
    Map<String, Paciente> pacientesTotales = new HashMap<>();
    PriorityQueue<Paciente> colaAtencion = new PriorityQueue<>();
    Map<String, AreaAtencion> areasAtencion = new HashMap<>();
    List<Paciente> pacientesAtendidos = new ArrayList<>();

    public Hospital() {
        areasAtencion.put("SAPU", new AreaAtencion("SAPU", 300));
        areasAtencion.put("urgencia_adulto", new AreaAtencion("urgencia_adulto", 300));
        areasAtencion.put("infantil", new AreaAtencion("infantil", 300));
    }

    public void registrarPaciente(Paciente p) {
        pacientesTotales.put(p.id, p);
        colaAtencion.offer(p);
        areasAtencion.get(p.area).ingresarPaciente(p);
    }

    public void reasignarCategoria(String id, int nuevaCategoria) {
        Paciente p = pacientesTotales.get(id);
        if (p != null) {
            p.registrarCambio("Cambio de categoria de " + p.categoria + " a " + nuevaCategoria);
            p.categoria = nuevaCategoria;
        }
    }

    public Paciente atenderSiguiente() {
        Paciente p = colaAtencion.poll();
        if (p != null) {
            p.estado = "atendido";
            pacientesAtendidos.add(p);
            areasAtencion.get(p.area).pacientesHeap.remove(p);
        }
        return p;
    }

    public Paciente atenderForzadoPorEspera(long minutoActual) {
        for (Paciente p : colaAtencion) {
            if (superaTiempoMaximo(p.categoria, p.tiempoEsperaActual(minutoActual))) {
                colaAtencion.remove(p);
                p.estado = "atendido";
                pacientesAtendidos.add(p);
                areasAtencion.get(p.area).pacientesHeap.remove(p);
                return p;
            }
        }
        return null;
    }

    public static boolean superaTiempoMaximo(int categoria, long espera) {
        return switch (categoria) {
            case 1 -> espera > 0;
            case 2 -> espera > 30;
            case 3 -> espera > 90;
            case 4 -> espera > 180;
            default -> false;
        };
    }

    public List<Paciente> getPacientesAtendidos() {
        return pacientesAtendidos;
    }

    public Paciente getPacientePorId(String id) {
        return pacientesTotales.get(id);
    }
}

class GeneradorPacientes {
    static String[] nombres = {"Ana", "Luis", "Pedro", "Clara", "María"};
    static String[] apellidos = {"Gómez", "Pérez", "Rojas", "Fuentes", "Silva"};
    static String[] areas = {"SAPU", "urgencia_adulto", "infantil"};
    static Random rand = new Random();

    public static List<Paciente> generarPacientes(int n, long minutoInicio) {
        List<Paciente> lista = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            String nombre = nombres[rand.nextInt(nombres.length)];
            String apellido = apellidos[rand.nextInt(apellidos.length)];
            String id = "P" + i;
            int categoria = generarCategoria();
            long llegada = minutoInicio + (i * 10);
            String area = areas[rand.nextInt(areas.length)];
            lista.add(new Paciente(nombre, apellido, id, categoria, llegada, area));
        }
        return lista;
    }

    private static int generarCategoria() {
        int r = rand.nextInt(100);
        if (r < 10) return 1;
        else if (r < 25) return 2;
        else if (r < 43) return 3;
        else if (r < 70) return 4;
        else return 5;
    }
}

class SimuladorUrgencia {
    public void simular(int pacientesPorDia) {
        System.out.println("Simulación general de jornada de 24 horas");
        Hospital hospital = new Hospital();
        List<Paciente> pacientes = GeneradorPacientes.generarPacientes(pacientesPorDia, 0);
        Queue<Paciente> colaLlegadas = new LinkedList<>(pacientes);
        int acumulador = 0;

        for (long minuto = 0; minuto < 1440; minuto++) {
            if (minuto % 10 == 0 && !colaLlegadas.isEmpty()) {
                hospital.registrarPaciente(colaLlegadas.poll());
                acumulador++;
                if (acumulador == 3) {
                    hospital.atenderSiguiente();
                    hospital.atenderSiguiente();
                    acumulador = 0;
                }
            }
            if (minuto % 15 == 0) {
                Paciente forzado = hospital.atenderForzadoPorEspera(minuto);
                if (forzado == null) hospital.atenderSiguiente();
            }
        }
        System.out.println("Pacientes atendidos: " + hospital.getPacientesAtendidos().size());
    }

    public void prueba1_seguimientoC4() {
        System.out.println("Prueba 1: Seguimiento paciente C4");
        Hospital hospital = new Hospital();
        Paciente p = new Paciente("Test", "Paciente", "P999", 4, 0, "SAPU");
        hospital.registrarPaciente(p);
        System.out.println("Paciente creado: " + p.nombre + " " + p.apellido + " (ID: " + p.id + ") - Categoría C" + p.categoria + ", Área: " + p.area);
        for (int m = 0; m <= 200; m += 15) {
            Paciente atendido = hospital.atenderForzadoPorEspera(m);
            if (atendido != null && atendido.id.equals("P999")) {
                System.out.println("Atendido en el minuto: " + m + " tras espera de " + atendido.tiempoEsperaActual(m) + " minutos.");
                return;
            }
        }
        System.out.println("No fue atendido dentro del tiempo máximo esperado para su categoría (180 min). Estado final: " + p.estado);
    }

    public void prueba2_promedioPorCategoria() {
        System.out.println("Prueba 2: Cálculo de tiempo promedio de espera por categoría");
        Map<Integer, Long> suma = new HashMap<>();
        Map<Integer, Integer> cuenta = new HashMap<>();
        for (int i = 0; i < 15; i++) {
            Hospital hospital = new Hospital();
            List<Paciente> pacientes = GeneradorPacientes.generarPacientes(144, 0);
            Queue<Paciente> llegadas = new LinkedList<>(pacientes);
            int nuevos = 0;
            for (int minuto = 0; minuto < 1440; minuto++) {
                if (minuto % 10 == 0 && !llegadas.isEmpty()) {
                    hospital.registrarPaciente(llegadas.poll());
                    nuevos++;
                    if (nuevos == 3) {
                        hospital.atenderSiguiente();
                        hospital.atenderSiguiente();
                        nuevos = 0;
                    }
                }
                if (minuto % 15 == 0) hospital.atenderSiguiente();
            }
            for (Paciente p : hospital.getPacientesAtendidos()) {
                suma.put(p.categoria, suma.getOrDefault(p.categoria, 0L) + p.tiempoEsperaActual(1440));
                cuenta.put(p.categoria, cuenta.getOrDefault(p.categoria, 0) + 1);
            }
        }
        for (int i = 1; i <= 5; i++) {
            long total = suma.getOrDefault(i, 0L);
            int c = cuenta.getOrDefault(i, 0);
            System.out.println("Categoría C" + i + " → Atendidos: " + c + ", Promedio de espera: " + (c > 0 ? total / c : 0) + " min");
        }
    }

    public void prueba3_saturacionSistema() {
        System.out.println("Prueba 3: Análisis bajo saturación del sistema (200 pacientes)");
        Hospital hospital = new Hospital();
        List<Paciente> pacientes = GeneradorPacientes.generarPacientes(200, 0);
        Queue<Paciente> cola = new LinkedList<>(pacientes);
        int nuevos = 0;
        Map<Integer, Integer> fueraTiempo = new HashMap<>();
        for (int m = 0; m < 1440; m++) {
            if (m % 10 == 0 && !cola.isEmpty()) {
                hospital.registrarPaciente(cola.poll());
                nuevos++;
                if (nuevos == 3) {
                    hospital.atenderSiguiente();
                    hospital.atenderSiguiente();
                    nuevos = 0;
                }
            }
            if (m % 15 == 0) hospital.atenderSiguiente();
        }
        for (Paciente p : hospital.getPacientesAtendidos()) {
            long espera = p.tiempoEsperaActual(1440);
            if (Hospital.superaTiempoMaximo(p.categoria, espera)) {
                fueraTiempo.put(p.categoria, fueraTiempo.getOrDefault(p.categoria, 0) + 1);
            }
        }
        fueraTiempo.forEach((cat, count) -> System.out.println("Categoría C" + cat + ": " + count + " pacientes excedieron su tiempo máximo de espera"));
    }

    public void prueba4_cambioCategoria() {
        System.out.println("Prueba 4: Reasignación de categoría con historial");
        Hospital hospital = new Hospital();
        Paciente p = new Paciente("Laura", "Test", "PX", 3, 0, "SAPU");
        hospital.registrarPaciente(p);
        System.out.println("Paciente inicial: " + p.nombre + " " + p.apellido + " (ID: " + p.id + ") - Categoría original: C" + p.categoria);
        hospital.reasignarCategoria("PX", 1);
        System.out.println("Historial tras reasignación: " + p.obtenerUltimoCambio());
        System.out.println("Categoría actual: C" + p.categoria);
    }

    public void simulacionExtra() {
        System.out.println("Simulación adicional (144 pacientes - jornada completa 24h)");
        Hospital hospital = new Hospital();
        List<Paciente> pacientes = GeneradorPacientes.generarPacientes(144, 0);
        Queue<Paciente> colaLlegadas = new LinkedList<>(pacientes);
        int acumulador = 0;

        Map<Integer, Long> tiemposCategoria = new HashMap<>();
        Map<Integer, Integer> cuentaCategoria = new HashMap<>();
        Map<Integer, Integer> fueraTiempo = new HashMap<>();

        for (long minuto = 0; minuto < 1440; minuto++) {
            if (minuto % 10 == 0 && !colaLlegadas.isEmpty()) {
                Paciente nuevo = colaLlegadas.poll();
                hospital.registrarPaciente(nuevo);
                acumulador++;
                if (acumulador == 3) {
                    hospital.atenderSiguiente();
                    hospital.atenderSiguiente();
                    acumulador = 0;
                }
            }

            if (minuto % 15 == 0) {
                Paciente forzado = hospital.atenderForzadoPorEspera(minuto);
                if (forzado == null) hospital.atenderSiguiente();
            }
        }

        for (Paciente p : hospital.getPacientesAtendidos()) {
            long espera = p.tiempoEsperaActual(1440);
            int cat = p.categoria;
            tiemposCategoria.put(cat, tiemposCategoria.getOrDefault(cat, 0L) + espera);
            cuentaCategoria.put(cat, cuentaCategoria.getOrDefault(cat, 0) + 1);
            if (Hospital.superaTiempoMaximo(cat, espera)) {
                fueraTiempo.put(cat, fueraTiempo.getOrDefault(cat, 0) + 1);
            }
        }

        System.out.println("Resultados de la simulación adicional:");
        for (int i = 1; i <= 5; i++) {
            int atendidos = cuentaCategoria.getOrDefault(i, 0);
            long totalTiempo = tiemposCategoria.getOrDefault(i, 0L);
            int fuera = fueraTiempo.getOrDefault(i, 0);
            System.out.println("Categoría C" + i +
                    " → Atendidos: " + atendidos +
                    ", Promedio espera: " + (atendidos > 0 ? totalTiempo / atendidos : 0) +
                    " min, Excedieron tiempo: " + fuera);
        }
        System.out.println("Total pacientes atendidos: " + hospital.getPacientesAtendidos().size());
    }
}
public class Main {
    public static void main(String[] args) {
        SimuladorUrgencia sim = new SimuladorUrgencia();
        sim.simular(144);
        sim.prueba1_seguimientoC4();
        sim.prueba2_promedioPorCategoria();
        sim.prueba3_saturacionSistema();
        sim.prueba4_cambioCategoria();
        sim.simulacionExtra();
    }
}**
