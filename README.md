# AvlJugadores
Registro de Jugadores
package com.avljugadores.controller;

import com.avltree.model.Node;
import com.avltree.model.Jugador;
import com.avltree.service.avljugadores;
import com.avltree.service.TreePersistenceService;
import com.avltree.util.TreeVisualizer;
import java.util.Scanner;

/**
 * Controlador principal que maneja el menú de la aplicación para árbol AVL de Jugadores
 */
public class AVLTreeController {
    private AVLTree<Jugador> tree;
    private TreePersistenceService<Jugador> persistenceService;
    private Scanner scanner;
    
    public AVLTreeController() {
        this.tree = new AVLTree<>();
        this.scanner = new Scanner(System.in);
        this.persistenceService = new TreePersistenceService<>(tree, clazz:Jugador.class);
    }
    
    /**
     * Método principal que ejecuta la aplicación
     */
    public void run() {
        showWelcomeMessage();
        
        try {
            // La conexión ya está validada, proceder a sincronizar datos
            System.out.println("🔄 Sincronizando datos existentes...");
            persistenceService.syncWithDatabase();
            System.out.println("✅ Datos sincronizados correctamente");
        } catch (Exception e) {
            System.out.println("⚠ Error al sincronizar datos: " + e.getMessage());
            System.out.println("Continuando con árbol vacío...");
        }
        
        boolean running = true;
        while (running) {
            showMainMenu();
            int option = getIntInput("Seleccione una opción: ");
            
            switch (option) {
                case 1:
                    insertJugador();
                    break;
                case 2:
                    updateJugador();
                    break;
                case 3:
                    searchJugador();
                    break;
                case 4:
                    deleteJugador();
                    break;
                case 5:
                    graphTree();
                    break;
                case 6:
                    databaseMenu();
                    break;
                case 7:
                    showTreeInfo();
                    break;
                case 8:
                    showAllJugador();
                    break;
                case 0:
                    running = false;
                    shutdown();
                    break;
                default:
                    System.out.println("Opción no válida. Por favor, intente de nuevo.");
            }
            
            if (running) {
                System.out.println("\nPresione Enter para continuar...");
                scanner.nextLine();
            }
        }
    }
    
    /**
     * Muestra el mensaje de bienvenida
     */
    private void showWelcomeMessage() {
        System.out.println("╔══════════════════════════════════════════════════════════════╗");
        System.out.println("║              ÁRBOL AVL DE JUGADORES CON MONGODB              ║");
        System.out.println("║             Gestión de Jugadores ordenadas por ID            ║");
        System.out.println("╚══════════════════════════════════════════════════════════════╝");
        System.out.println();
    }
    
    /**
     * Muestra el menú principal
     */
    private void showMainMenu() {
        System.out.println("\n" + "=".repeat(60));
        System.out.println("                    MENÚ PRINCIPAL");
        System.out.println("=".repeat(60));
        System.out.println("1. Insertar Jugador");
        System.out.println("2. Actualizar Jugador");
        System.out.println("3. Buscar Jugador por ID");
        System.out.println("4. Eliminar Jugador");
        System.out.println("5. Graficar árbol");
        System.out.println("6. Operaciones de base de datos");
        System.out.println("7. Información del árbol");
        System.out.println("8. Mostrar todos los Jugadores");
        System.out.println("0. Salir");
        System.out.println("=".repeat(60));
    }
    
    /**
     * Maneja la inserción de un jugador
     */
    private void insertJugador() {
        System.out.println("\n=== INSERTAR JUGADOR ===");
        
        try {
            String ID = getStringInput("Ingrese el ID del Jugador: ");
            String nombre = getStringInput("Ingrese el nombre: ");
            String apellido = getStringInput("Ingrese el apellido: ");
            String deporte = getStringInput("Ingrese el deporte: ");
            String equipo = getStringInput("Ingrese el equipo: ");
            int edad = getIntInput("Ingrese la edad: ");
            
            Jugador Jugador = new Jugador(nombre, apellido, edad, ID);
            
            if (!Jugador.isValid()) {
                System.out.println("Datos de jugador inválidos. Verifique que:");
                System.out.println("   - Nombre y apellido no estén vacíos");
                System.out.println("   - Deporte no esté vacío");
                System.out.println("   - Equipo no esté vacío");
                System.out.println("   - Edad esté entre 1 y 149 años");
                System.out.println("   - Id tenga al menos 5 caracteres");
                return;
            }
            
            tree.insert(jugador);
            System.out.println("✓ Jugador insertado exitosamente");
            System.out.println("✓ " + Jugador.getDetalle());
            
            // Guardar en base de datos si es posible
            Node<Jugador> insertedNode = tree.search(jugador);
            if (insertedNode != null) {
                persistenceService.saveNode(insertedNode);
            }
            
        } catch (Exception e) {
            System.out.println("Error al insertar el jugador: " + e.getMessage());
        }
    }
    
    /**
     * Maneja la actualización de una persona
     */
    private void updateJugador() {
        System.out.println("\n=== ACTUALIZAR JUGADOR ===");
        
        if (tree.isEmpty()) {
            System.out.println("⚠ El árbol está vacío");
            return;
        }
        
        try {
            String ID = getStringInput("Ingrese el ID del Jugador a actualizar: ");
            
            // Crear persona temporal para búsqueda
            Jugador searchJugador = new Jugador("", "", 0, ID);
            Node<Jugador> node = tree.search(searchJugador);
            
            if (node == null) {
                System.out.println("❌ No se encontró el jugador con ID " + Id);
                return;
            }
            
            Jugador currentJugador = node.getData();
            System.out.println("Jugador actual: " + currentJugador.getDetalle());
            System.out.println();
            
            System.out.println("Ingrese los nuevos datos (presione Enter para mantener el valor actual):");
            
            String nuevoNombre = getStringInputOptional("Nuevo nombre [" + currentJugador.getNombre() + "]: ");
            if (nuevoNombre.isEmpty()) {
                nuevoNombre = currentJugador.getNombre();
            }
            
            String nuevoApellido = getStringInputOptional("Nuevo apellido [" + currentJugador.getApellido() + "]: ");
            if (nuevoApellido.isEmpty()) {
                nuevoApellido = currentJugador.getApellido();
            }

            String nuevoDeporte = getStringInputOptional("Nuevo deporte [" + currentJugador.getDeporte() + "]: ");
            if (nuevoDeporte.isEmpty()) {
                nuevoDeporte = currentJugador.getDeporte();
            }

            String nuevoEquipo = getStringInputOptional("Nuevo equipo [" + currentJugador.getEquipo() + "]: ");
            if (nuevoEquipo.isEmpty()) {
                nuevoEquipo = currentJugador.getEquipo();
            }
            
            String edadStr = getStringInputOptional("Nueva edad [" + currentJugador.getEdad() + "]: ");
            int nuevaEdad = currentJugador.getEdad();
            if (!edadStr.isEmpty()) {
                try {
                    nuevaEdad = Integer.parseInt(edadStr);
                } catch (NumberFormatException e) {
                    System.out.println("❌ Edad inválida, manteniendo valor actual");
                    nuevaEdad = currentJugador.getEdad();
                }
            }
            
            String nuevoID = getStringInputOptional("Nuevo ID [" + currentJugador.getID() + "]: ");
            if (nuevoID.isEmpty()) {
                nuevoID = currentJugador.getID();
            }
            
            Jugador nuevoJugador = new Jugador(nuevoNombre, nuevoApellido, nuevoDeporte, nuevoEquipo, nuevaEdad, nuevoID);
            
            if (!nuevaJugador.isValid()) {
                System.out.println("❌ Los nuevos datos son inválidos");
                return;
            }
            
            boolean updated = tree.update(currentJugador, nuevaJugador);
            if (updated) {
                System.out.println("✓ Jugador actualizado exitosamente");
                System.out.println("✓ " + nuevaJugador.getDetalle());
                
                // Actualizar en base de datos
                Node<Jugador> updatedNode = tree.search(nuevoJugador);
                if (updatedNode != null) {
                    persistenceService.saveNode(updatedNode);
                }
            } else {
                System.out.println("❌ No se pudo actualizar el jugador");
            }
            
        } catch (Exception e) {
            System.out.println("❌ Error al actualizar el jugador: " + e.getMessage());
        }
    }
    
    /**
     * Maneja la búsqueda de un jugador
     */
    private void searchJugador() {
        System.out.println("\n=== BUSCAR JUGADOR ===");
        
        if (tree.isEmpty()) {
            System.out.println("⚠ El árbol está vacío");
            return;
        }
        
        try {
            String ID = getStringInput("Ingrese el ID a buscar: ");
            
            // Crear jugador temporal para búsqueda
            Jugador searchJugador = new Jugador("", "", 0, ID);
            Node<Jugador> node = tree.search(searchJugador);
            
            if (node != null) {
                Jugador Jugador = node.getData();
                System.out.println("✓ Jugador encontrado:");
                System.out.println("  " + jugador.getDetalle());
                System.out.println("  Altura en árbol: " + node.getHeight());
            } else {
                System.out.println("❌ No se encontró el jugador con el ID " + Id);
            }
            
        } catch (Exception e) {
            System.out.println("❌ Error al buscar el jugador: " + e.getMessage());
        }
    }
    
    /**
     * Maneja la eliminación de un jugador
     */
    private void deleteJugador() {
        System.out.println("\n=== ELIMINAR JUGADOR ===");
        
        if (tree.isEmpty()) {
            System.out.println("⚠ El árbol está vacío");
            return;
        }
        
        try {
            String dpi = getStringInput("Ingrese el Id del jugador a eliminar: ");
            
            // Crear jugador temporal para búsqueda
            Jugador searchJugador = new Jugador("", "", 0, ID);
            Node<Jugador> node = tree.search(searchjugador);
            
            if (node == null) {
                System.out.println("❌ No se encontró el jugador con el Id " + Id);
                return;
            }
            
            Jugador jugador = node.getData();
            System.out.println("Jugador a eliminar: " + jugador.getDetalle());
            String confirm = getStringInput("¿Está seguro? (s/n): ");
            
            if (confirm.toLowerCase().startsWith("s")) {
                tree.delete(jugador);
                System.out.println("✓ Jugador eliminado exitosamente");
                
                // Eliminar de base de datos
                persistenceService.deleteNode(jugador);
                
            } else {
                System.out.println("Operación cancelada");
            }
            
        } catch (Exception e) {
            System.out.println("❌ Error al eliminar el jugador: " + e.getMessage());
        }
    }
    
    /**
     * Maneja la visualización del árbol
     */
    private void graphTree() {
        System.out.println("\n=== GRAFICAR ÁRBOL ===");
        
        if (tree.isEmpty()) {
            System.out.println("⚠ El árbol está vacío");
            return;
        }
        
        boolean showingGraph = true;
        while (showingGraph) {
            TreeVisualizer.showVisualizationMenu(tree.getRoot());
            int option = getIntInput("Seleccione una opción: ");
            
            if (option == 0) {
                showingGraph = false;
            } else {
                TreeVisualizer.executeVisualization(tree.getRoot(), option);
                if (option != 4) { // Si no es "todas las visualizaciones"
                    System.out.println("\nPresione Enter para continuar...");
                    scanner.nextLine();
                }
            }
        }
    }
    
    /**
     * Muestra todos los jugadores ordenados por ID
     */
    private void showAllJugador() {
        System.out.println("\n=== TODOS LOS JUGADORES (ORDENADOS POR ID) ===");
        
        if (tree.isEmpty()) {
            System.out.println("⚠ No hay jugadores registrados");
            return;
        }
        
        var nodes = tree.inorderTraversal();
        System.out.println("Total de jugdores: " + nodes.size());
        System.out.println();
        
        int count = 1;
        for (Node<Jugador> node : nodes) {
            Jugador jugador = node.getData();
            System.out.printf("%3d. %s\n", count++, jugador.getDetalle());
        }
    }
    
    /**
     * Muestra el menú de operaciones de base de datos
     */
    private void databaseMenu() {
        boolean inDatabaseMenu = true;
        
        while (inDatabaseMenu) {
            System.out.println("\n=== OPERACIONES DE BASE DE DATOS ===");
            System.out.println("1. Guardar árbol completo");
            System.out.println("2. Cargar árbol desde BD");
            System.out.println("3. Sincronizar con BD");
            System.out.println("4. Estadísticas de BD");
            System.out.println("5. Verificar integridad");
            System.out.println("6. Limpiar base de datos");
            System.out.println("0. Regresar");
            
            int option = getIntInput("Seleccione una opción: ");
            
            switch (option) {
                case 1:
                    persistenceService.saveTree();
                    break;
                case 2:
                    persistenceService.loadTree();
                    break;
                case 3:
                    persistenceService.syncWithDatabase();
                    break;
                case 4:
                    persistenceService.printDatabaseStats();
                    break;
                case 5:
                    persistenceService.verifyIntegrity();
                    break;
                case 6:
                    String confirm = getStringInput("¿Está seguro de limpiar la BD? (s/n): ");
                    if (confirm.toLowerCase().startsWith("s")) {
                        persistenceService.clearDatabase();
                    }
                    break;
                case 0:
                    inDatabaseMenu = false;
                    break;
                default:
                    System.out.println("❌ Opción no válida");
            }
            
            if (inDatabaseMenu && option != 0) {
                System.out.println("\nPresione Enter para continuar...");
                scanner.nextLine();
            }
        }
    }
    
    /**
     * Muestra información general del árbol
     */
    private void showTreeInfo() {
        System.out.println("\n=== INFORMACIÓN DEL ÁRBOL ===");
        
        if (tree.isEmpty()) {
            System.out.println("⚠ El árbol está vacío");
        } else {
            TreeVisualizer.printTreeInfo(tree.getRoot());
        }
    }
    
    /**
     * Maneja el cierre de la aplicación
     */
    private void shutdown() {
        System.out.println("\n=== CERRANDO APLICACIÓN ===");
        
        try {
            // Guardar cambios antes de salir
            if (!tree.isEmpty()) {
                String save = getStringInput("¿Desea guardar los cambios en la base de datos? (s/n): ");
                if (save.toLowerCase().startsWith("s")) {
                    persistenceService.saveTree();
                }
            }
            
            System.out.println("¡Gracias por usar el sistema de Gestión de Jugadores con Árbol AVL!");
            
        } catch (Exception e) {
            System.out.println("Error durante el cierre: " + e.getMessage());
        } finally {
            scanner.close();
        }
    }
    
    /**
     * Obtiene entrada de entero del usuario con validación
     */
    private int getIntInput(String prompt) {
        while (true) {
            try {
                System.out.print(prompt);
                String input = scanner.nextLine().trim();
                return Integer.parseInt(input);
            } catch (NumberFormatException e) {
                System.out.println("❌ Por favor, ingrese un número válido.");
            }
        }
    }
    
    /**
     * Obtiene entrada de cadena del usuario
     */
    private String getStringInput(String prompt) {
        System.out.print(prompt);
        return scanner.nextLine().trim();
    }
    
    /**
     * Obtiene entrada de cadena opcional del usuario
     */
    private String getStringInputOptional(String prompt) {
        System.out.print(prompt);
        return scanner.nextLine().trim();
    }
}
