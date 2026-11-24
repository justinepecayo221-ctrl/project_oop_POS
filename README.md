package Project;

import javax.swing.*;
import javax.swing.border.*;
import java.awt.*;
import java.awt.event.*;
import java.awt.geom.*;
import java.io.*;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import javax.swing.table.DefaultTableModel;
import java.time.LocalDate;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.*;
import java.util.List;

public class coffee_shop extends JFrame {

    private static final Color CREAM  = new Color(255, 238, 218);
    private static final Color BROWN  = new Color(105, 48, 28);
    private static final Color DARK   = new Color(65, 29, 15);
    private static final Color BORDER = new Color(120, 65, 42);
    private static final Color HINT   = new Color(130, 105, 88);

    
    private static final String MANAGER_CODE = "manager123";

    private static final File USER_FILE = new File("users.txt");
    private final Map<String, String> users = new HashMap<>();  
    private final Map<String, String> roles = new HashMap<>();  

    public coffee_shop() {

        loadUsers();

        setTitle("Coffee Shop Portal");
        setSize(1672, 941);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setLayout(null);

        // LEFT BACKGROUND
        java.net.URL bgURL = getClass().getResource("/bgcoffee.png");

        if (bgURL == null) {
            System.out.println("ERROR: bgcoffee.png not found!");
        } else {
            ImageIcon bg = new ImageIcon(bgURL);
            Image scaled = bg.getImage().getScaledInstance(960, 886, Image.SCALE_SMOOTH);

            JLabel background = new JLabel(new ImageIcon(scaled));
            background.setBounds(0, 0, 960, 886);
            add(background);
        }

        // RIGHT PANEL
        JPanel panel = new JPanel(null) {
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);

                Graphics2D g2 = (Graphics2D) g;
                g2.setRenderingHint(RenderingHints.KEY_ANTIALIASING,
                                    RenderingHints.VALUE_ANTIALIAS_ON);
                g2.setColor(CREAM);

                Path2D shape = new Path2D.Double();
                shape.moveTo(30, 0);
                shape.curveTo(-10, 200, -10, 650, 30, 886);
                shape.lineTo(712, 886);
                shape.lineTo(712, 0);
                shape.closePath();

                g2.fill(shape);
            }
        };

        panel.setBounds(920, 0, 752, 886);
        panel.setOpaque(false);
        add(panel);

        // COFFEE ICON
        java.net.URL iconURL = getClass().getResource("/Iconbg.png");

        if (iconURL == null) {
            System.out.println("ERROR: Iconbg.png not found!");
        } else {
            ImageIcon coffee = new ImageIcon(iconURL);
            Image coffeeImage = coffee.getImage().getScaledInstance(110, 110, Image.SCALE_SMOOTH);

            JLabel coffeeLabel = new JLabel(new ImageIcon(coffeeImage));
            coffeeLabel.setBounds(285, 65, 110, 110);
            panel.add(coffeeLabel);
        }

        // TITLE
        JLabel title = new JLabel("Login to manage");
        title.setFont(new Font("Arial", Font.BOLD, 34));
        title.setForeground(DARK);
        title.setHorizontalAlignment(SwingConstants.CENTER);
        title.setBounds(130, 210, 430, 55);
        panel.add(title);

        // USERNAME 
        JTextField username = new JTextField("Username");
        username.setFont(new Font("Arial", Font.PLAIN, 27));
        username.setForeground(HINT);
        username.setBounds(100, 290, 515, 72);
        username.setBorder(fieldBorder());
        addPlaceholder(username, "Username");
        panel.add(username);

        // PASSWORD
        JPasswordField password = new JPasswordField();
        password.setFont(new Font("Arial", Font.PLAIN, 27));
        password.setEchoChar('\u25CF');
        password.setBorder(fieldBorder());
        password.setBounds(100, 385, 515, 72);
        panel.add(password);

        // LOGIN BUTTON
        JButton login = new JButton("Log In");
        login.setFont(new Font("Arial", Font.BOLD, 27));
        login.setForeground(Color.WHITE);
        login.setBackground(BROWN);
        login.setBounds(140, 500, 435, 75);
        login.setFocusPainted(false);
        login.setBorder(new LineBorder(BROWN, 1, true));
        login.setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
        panel.add(login);

        // FORGOT PASSWORD
        JButton forgot = new JButton("Forgot password?");
        forgot.setFont(new Font("Arial", Font.BOLD, 22));
        forgot.setForeground(Color.BLACK);
        forgot.setBorderPainted(false);
        forgot.setContentAreaFilled(false);
        forgot.setBounds(200, 590, 320, 45);
        forgot.setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
        panel.add(forgot);

        // OR
        JLabel or = new JLabel("or");
        or.setFont(new Font("Arial", Font.BOLD, 22));
        or.setHorizontalAlignment(SwingConstants.CENTER);
        or.setBounds(285, 645, 100, 40);
        panel.add(or);

        // REGISTER BUTTON
        JButton register = new JButton("Register an account");
        register.setFont(new Font("Arial", Font.BOLD, 22));
        register.setForeground(DARK);
        register.setBackground(CREAM);
        register.setBounds(135, 690, 440, 65);
        register.setBorder(new LineBorder(new Color(85, 42, 25), 2, true));
        register.setFocusPainted(false);
        register.setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
        panel.add(register);

        // ===== ACTIONS =====

        // LOGIN
        ActionListener doLogin = e -> {
            String user = username.getText().trim();
            String pass = new String(password.getPassword());

            if (user.isEmpty() || user.equals("Username") || pass.isEmpty()) {
                JOptionPane.showMessageDialog(this,
                    "Please enter your username and password.",
                    "Missing information", JOptionPane.WARNING_MESSAGE);
                return;
            }

            String saved = users.get(user.toLowerCase());
            if (saved != null && saved.equals(hash(pass))) {
                String role = roles.getOrDefault(user.toLowerCase(), "cashier");
                dispose();
                new CoffeeShopPOS(role, user);  
            } else {
                JOptionPane.showMessageDialog(this,
                    "Incorrect username or password.",
                    "Login failed", JOptionPane.ERROR_MESSAGE);
                password.setText("");
            }
        };
        login.addActionListener(doLogin);
        password.addActionListener(doLogin); 

        // REGISTER
        register.addActionListener(e -> showRegisterDialog(username));

        // FORGOT PASSWORD
        forgot.addActionListener(e -> showResetDialog());

        setVisible(true);
    }

    //  REGISTER DIALOG

    private void showRegisterDialog(JTextField loginUsernameField) {

        JDialog dialog = new JDialog(this, "Register an account", true);
        dialog.setSize(520, 690);
        dialog.setLocationRelativeTo(this);
        dialog.setResizable(false);
        dialog.setLayout(null);
        dialog.getContentPane().setBackground(CREAM);

        JLabel head = new JLabel("Create your account");
        head.setFont(new Font("Arial", Font.BOLD, 28));
        head.setForeground(DARK);
        head.setHorizontalAlignment(SwingConstants.CENTER);
        head.setBounds(40, 25, 440, 45);
        dialog.add(head);

        JTextField newUser = new JTextField("Username");
        styleField(newUser, 95);
        newUser.setForeground(HINT);
        addPlaceholder(newUser, "Username");
        dialog.add(newUser);

        JPasswordField newPass = new JPasswordField();
        styleField(newPass, 175);
        newPass.setEchoChar('\u25CF');
        dialog.add(newPass);

        JLabel passHint = new JLabel("Password (at least 6 characters)");
        passHint.setFont(new Font("Arial", Font.PLAIN, 13));
        passHint.setForeground(HINT);
        passHint.setBounds(55, 238, 400, 18);
        dialog.add(passHint);

        JPasswordField confirmPass = new JPasswordField();
        styleField(confirmPass, 265);
        confirmPass.setEchoChar('\u25CF');
        dialog.add(confirmPass);

        JLabel confirmHint = new JLabel("Confirm password");
        confirmHint.setFont(new Font("Arial", Font.PLAIN, 13));
        confirmHint.setForeground(HINT);
        confirmHint.setBounds(55, 328, 400, 18);
        dialog.add(confirmHint);

        // ROLE
        JLabel roleLabel = new JLabel("I am a:");
        roleLabel.setFont(new Font("Arial", Font.BOLD, 18));
        roleLabel.setForeground(DARK);
        roleLabel.setBounds(60, 362, 80, 30);
        dialog.add(roleLabel);

        JRadioButton cashierBtn = new JRadioButton("Cashier", true);
        JRadioButton managerBtn = new JRadioButton("Manager");
        for (JRadioButton r : new JRadioButton[]{cashierBtn, managerBtn}) {
            r.setFont(new Font("Arial", Font.BOLD, 18));
            r.setForeground(DARK);
            r.setOpaque(false);
            r.setFocusPainted(false);
        }
        cashierBtn.setBounds(145, 362, 120, 30);
        managerBtn.setBounds(275, 362, 140, 30);
        ButtonGroup roleGroup = new ButtonGroup();
        roleGroup.add(cashierBtn);
        roleGroup.add(managerBtn);
        dialog.add(cashierBtn);
        dialog.add(managerBtn);

        // Manager access code 
        JPasswordField codeField = new JPasswordField();
        styleField(codeField, 405);
        codeField.setBounds(60, 405, 400, 52);
        codeField.setEchoChar('\u25CF');
        codeField.setVisible(false);
        dialog.add(codeField);

        JLabel codeHint = new JLabel("Manager access code");
        codeHint.setFont(new Font("Arial", Font.PLAIN, 13));
        codeHint.setForeground(HINT);
        codeHint.setBounds(55, 460, 400, 18);
        codeHint.setVisible(false);
        dialog.add(codeHint);

        ActionListener roleChanged = e -> {
            boolean mgr = managerBtn.isSelected();
            codeField.setVisible(mgr);
            codeHint.setVisible(mgr);
        };
        cashierBtn.addActionListener(roleChanged);
        managerBtn.addActionListener(roleChanged);

        JLabel error = new JLabel(" ");
        error.setFont(new Font("Arial", Font.BOLD, 14));
        error.setForeground(new Color(180, 30, 30));
        error.setHorizontalAlignment(SwingConstants.CENTER);
        error.setBounds(40, 490, 440, 24);
        dialog.add(error);

        JButton create = new JButton("Create Account");
        create.setFont(new Font("Arial", Font.BOLD, 22));
        create.setForeground(Color.WHITE);
        create.setBackground(BROWN);
        create.setFocusPainted(false);
        create.setBorder(new LineBorder(BROWN, 1, true));
        create.setBounds(60, 525, 400, 60);
        create.setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
        dialog.add(create);

        JButton cancel = new JButton("Back to login");
        cancel.setFont(new Font("Arial", Font.BOLD, 18));
        cancel.setForeground(DARK);
        cancel.setBorderPainted(false);
        cancel.setContentAreaFilled(false);
        cancel.setBounds(135, 595, 250, 35);
        cancel.setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
        dialog.add(cancel);

        ActionListener doCreate = e -> {
            String user = newUser.getText().trim();
            String pass = new String(newPass.getPassword());
            String confirm = new String(confirmPass.getPassword());
            boolean wantsManager = managerBtn.isSelected();

            if (user.isEmpty() || user.equals("Username")) {
                error.setText("Please enter a username.");
            } else if (user.length() < 3) {
                error.setText("Username must be at least 3 characters.");
            } else if (user.contains(":") || user.contains(" ")) {
                error.setText("Username cannot contain spaces or ':'.");
            } else if (users.containsKey(user.toLowerCase())) {
                error.setText("That username is already taken.");
            } else if (pass.length() < 6) {
                error.setText("Password must be at least 6 characters.");
            } else if (!pass.equals(confirm)) {
                error.setText("Passwords do not match.");
            } else if (wantsManager && !new String(codeField.getPassword()).equals(MANAGER_CODE)) {
                error.setText("Invalid manager access code.");
            } else {
                users.put(user.toLowerCase(), hash(pass));
                roles.put(user.toLowerCase(), wantsManager ? "manager" : "cashier");
                saveUsers();
                JOptionPane.showMessageDialog(dialog,
                    "Account created as " + (wantsManager ? "Manager" : "Cashier") + "! You can now log in.",
                    "Success", JOptionPane.INFORMATION_MESSAGE);

                // Fill in the login username for convenience
                loginUsernameField.setText(user);
                loginUsernameField.setForeground(Color.BLACK);
                dialog.dispose();
            }
        };
        create.addActionListener(doCreate);
        confirmPass.addActionListener(doCreate); // Enter key submits
        cancel.addActionListener(e -> dialog.dispose());

        dialog.setVisible(true);
    }

    //  FORGOT PASSWORD 

    private void showResetDialog() {
        String user = JOptionPane.showInputDialog(this,
            "Enter your username:", "Reset password", JOptionPane.QUESTION_MESSAGE);

        if (user == null) return; // cancelled
        user = user.trim().toLowerCase();

        if (!users.containsKey(user)) {
            JOptionPane.showMessageDialog(this,
                "No account found with that username.",
                "Not found", JOptionPane.ERROR_MESSAGE);
            return;
        }

        JPasswordField p1 = new JPasswordField();
        JPasswordField p2 = new JPasswordField();
        Object[] form = { "New password (min 6 characters):", p1, "Confirm new password:", p2 };

        int result = JOptionPane.showConfirmDialog(this, form,
            "Reset password", JOptionPane.OK_CANCEL_OPTION, JOptionPane.PLAIN_MESSAGE);

        if (result != JOptionPane.OK_OPTION) return;

        String a = new String(p1.getPassword());
        String b = new String(p2.getPassword());

        if (a.length() < 6) {
            JOptionPane.showMessageDialog(this, "Password must be at least 6 characters.",
                "Error", JOptionPane.ERROR_MESSAGE);
        } else if (!a.equals(b)) {
            JOptionPane.showMessageDialog(this, "Passwords do not match.",
                "Error", JOptionPane.ERROR_MESSAGE);
        } else {
            users.put(user, hash(a));
            saveUsers();
            JOptionPane.showMessageDialog(this, "Password updated. You can now log in.",
                "Success", JOptionPane.INFORMATION_MESSAGE);
        }
    }

    //  HELPERS

    private Border fieldBorder() {
        return new CompoundBorder(
            new LineBorder(BORDER, 2, true),
            new EmptyBorder(10, 25, 10, 15));
    }

    private void styleField(JTextField f, int y) {
        f.setFont(new Font("Arial", Font.PLAIN, 24));
        f.setBorder(fieldBorder());
        f.setBounds(60, y, 400, 62);
    }

    private void addPlaceholder(JTextField field, String placeholder) {
        field.addFocusListener(new FocusAdapter() {
            public void focusGained(FocusEvent e) {
                if (field.getText().equals(placeholder)) {
                    field.setText("");
                    field.setForeground(Color.BLACK);
                }
            }
            public void focusLost(FocusEvent e) {
                if (field.getText().trim().isEmpty()) {
                    field.setText(placeholder);
                    field.setForeground(HINT);
                }
            }
        });
    }

    private static String hash(String text) {
        try {
            MessageDigest md = MessageDigest.getInstance("SHA-256");
            byte[] bytes = md.digest(text.getBytes(StandardCharsets.UTF_8));
            StringBuilder sb = new StringBuilder();
            for (byte b : bytes) sb.append(String.format("%02x", b));
            return sb.toString();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    private void loadUsers() {
        if (!USER_FILE.exists()) return;
        try (BufferedReader br = new BufferedReader(new FileReader(USER_FILE))) {
            String line;
            while ((line = br.readLine()) != null) {
                String[] p = line.split(":");
                if (p.length >= 2 && !p[0].isEmpty()) {
                    users.put(p[0], p[1]);
                    roles.put(p[0], p.length >= 3 ? p[2] : "cashier");
                }
            }
        } catch (IOException e) {
            System.out.println("Could not load users: " + e.getMessage());
        }
    }

    private void saveUsers() {
        try (PrintWriter pw = new PrintWriter(new FileWriter(USER_FILE))) {
            for (Map.Entry<String, String> en : users.entrySet()) {
                pw.println(en.getKey() + ":" + en.getValue() + ":" + roles.getOrDefault(en.getKey(), "cashier"));
            }
        } catch (IOException e) {
            JOptionPane.showMessageDialog(this, "Could not save account: " + e.getMessage());
        }
    }

    public static void main(String[] args) {
        try { UIManager.setLookAndFeel(UIManager.getCrossPlatformLookAndFeelClassName()); } catch (Exception ignored) { }
        SwingUtilities.invokeLater(() -> new coffee_shop());
    }
}

class CoffeeShopPOS extends JFrame {

    //colors 
    static final Color DARK = new Color(0x2B1D1A), TAN = new Color(0xC49A6C), BG = new Color(0xD9D9D9),
            RED = new Color(0xE02828), GREEN = new Color(0x2E8B57), CARD = new Color(0xF2F2F2);
    static final Font BOLD = new Font("SansSerif", Font.BOLD, 14);
    static final String[] CATS = {"Coffee", "Non-coffee", "Food"};
    static final Map<String, Double> SIZE_ADD = new LinkedHashMap<>();
    static final Map<String, Double> DISC = new LinkedHashMap<>();
    static {
        SIZE_ADD.put("Tall", 0.0); SIZE_ADD.put("Grande", 10.0); SIZE_ADD.put("Venti", 20.0);
        DISC.put("None", 0.0); DISC.put("First-Time", 0.10); DISC.put("Student", 0.15);
        DISC.put("Loyalty", 0.10); DISC.put("Combos", 0.05);
    }

    //data 
    static class Product {
        String name, cat; double price; ImageIcon img;
        Product(String n, double p, String c) { name = n; price = p; cat = c; }
    }
    static class Line {
        String label, base; int qty; double unit;
        Line(String label, String base, int qty, double unit) { this.label = label; this.base = base; this.qty = qty; this.unit = unit; }
    }
    static class Order {
        String code, type, payment, sugar, size, discName;
        LocalDateTime time = LocalDateTime.now();
        List<Line> lines = new ArrayList<>();
        double subtotal, discount, total, paid;
    }

    final List<Product> products = new ArrayList<>();
    final Map<Product, Integer> cart = new LinkedHashMap<>();
    final Map<String, Order> orders = new LinkedHashMap<>();
    final String role, username;   // role is "manager" or "cashier"
    String employeeName;

    //screens
    final CardLayout layout = new CardLayout();
    final JPanel root = new JPanel(layout);
    Dashboard empDash, mgrDash;
    final JPanel orderPanel = new JPanel(), receiptPanel = new JPanel(), salesHolder = new JPanel();
    JPanel trackPanel;

    public CoffeeShopPOS(String role, String username) {
        super("Coffeum Est Vita - Point of Sale");
        this.role = role;
        this.username = username;
        this.employeeName = username.toUpperCase();
        seed();
        empDash = new Dashboard(false);
        mgrDash = new Dashboard(true);
        buildTrack();
        root.add(empDash, "employee");
        root.add(mgrDash, "manager");
        root.add(orderPanel, "order");
        root.add(trackPanel, "track");
        root.add(receiptPanel, "receipt");
        root.add(salesHolder, "sales");
        setContentPane(root);
        setSize(1100, 720);
        setLocationRelativeTo(null);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        goTo(role.equals("manager") ? "manager" : "employee");
        setVisible(true);
    }

    void seed() {
        String[][] d = {
                {"Americano", "90", "Coffee"}, {"Latte", "110", "Coffee"}, {"Cappuccino", "110", "Coffee"},
                {"Espresso", "80", "Coffee"}, {"Mocha", "120", "Coffee"}, {"Caramel Macchiato", "130", "Coffee"},
                {"Matcha Latte", "120", "Non-coffee"}, {"Hot Chocolate", "100", "Non-coffee"},
                {"Milk Tea", "95", "Non-coffee"}, {"Strawberry Milk", "105", "Non-coffee"},
                {"Croissant", "85", "Food"}, {"Cheesecake", "140", "Food"},
                {"Club Sandwich", "150", "Food"}, {"Blueberry Muffin", "70", "Food"}};
        for (String[] r : d) products.add(new Product(r[0], Double.parseDouble(r[1]), r[2]));
    }

    void goTo(String card) { layout.show(root, card); }

    static String money(double v) { return String.format("PHP %,.2f", v); }

    static JButton btn(String text, Color bg, Color fg) {
        JButton b = new JButton(text);
        b.setBackground(bg); b.setForeground(fg); b.setFont(BOLD);
        b.setFocusPainted(false); b.setOpaque(true); b.setBorderPainted(false);
        b.setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
        b.setBorder(new EmptyBorder(8, 16, 8, 16));
        return b;
    }

    static JLabel titleLabel(String t) {
        JLabel l = new JLabel(t); l.setForeground(Color.WHITE); l.setFont(new Font("SansSerif", Font.PLAIN, 24)); return l;
    }

    static JPanel header(JLabel title, boolean center, JComponent right) {
        JPanel h = new JPanel(new BorderLayout());
        h.setBackground(DARK); h.setBorder(new EmptyBorder(10, 20, 10, 20));
        if (center) title.setHorizontalAlignment(SwingConstants.CENTER);
        h.add(title, center ? BorderLayout.CENTER : BorderLayout.WEST);
        if (right != null) {
            JPanel w = new JPanel(new FlowLayout(FlowLayout.RIGHT, 8, 0)); w.setOpaque(false); w.add(right);
            h.add(w, BorderLayout.EAST);
            if (center) {
                JPanel spacer = new JPanel(); spacer.setOpaque(false); spacer.setPreferredSize(w.getPreferredSize());
                h.add(spacer, BorderLayout.WEST);
            }
        }
        return h;
    }

    static ImageIcon scale(ImageIcon i, int w, int h) {
        return new ImageIcon(i.getImage().getScaledInstance(w, h, Image.SCALE_SMOOTH));
    }

    static String sel(ButtonGroup g) { return g.getSelection().getActionCommand(); }

    static double unit(Product p, String size) { return p.price + (p.cat.equals("Food") ? 0 : SIZE_ADD.get(size)); }

    class Avatar extends JButton {
        Avatar() {
            setPreferredSize(new Dimension(46, 46));
            setContentAreaFilled(false); setBorderPainted(false); setFocusPainted(false);
            setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
            JPopupMenu m = new JPopupMenu();
            JMenuItem who = new JMenuItem(username + " (" + (role.equals("manager") ? "Manager" : "Cashier") + ")");
            who.setEnabled(false);
            JMenuItem out = new JMenuItem("Log out");
            out.addActionListener(e -> {
                if (JOptionPane.showConfirmDialog(CoffeeShopPOS.this, "Log out of the system?", "Log out",
                        JOptionPane.YES_NO_OPTION) == JOptionPane.YES_OPTION) {
                    CoffeeShopPOS.this.dispose();
                    SwingUtilities.invokeLater(() -> new coffee_shop());
                }
            });
            m.add(who); m.addSeparator(); m.add(out);
            addActionListener(e -> m.show(this, -100, getHeight()));
        }
        @Override protected void paintComponent(Graphics g0) {
            Graphics2D g = (Graphics2D) g0.create();
            g.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
            int w = getWidth(), h = getHeight();
            g.setColor(new Color(0xE6E6E6)); g.fillRoundRect(0, 0, w - 1, h - 1, 10, 10);
            g.setColor(new Color(0x9A9A9A)); g.fillOval(w / 2 - 8, 8, 16, 16);
            g.setClip(0, 0, w, h); g.fillArc(w / 2 - 14, 28, 28, 28, 0, 180);
            g.dispose();
        }
    }
    
    static class IconButton extends JButton {
        final boolean cart; int badge;
        IconButton(boolean cart) {
            this.cart = cart;
            setPreferredSize(new Dimension(50, 48));
            setContentAreaFilled(false); setBorderPainted(false); setFocusPainted(false);
            setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
            setToolTipText(cart ? "View order" : "Track order");
        }
        void setBadge(int n) { badge = n; repaint(); }
        @Override protected void paintComponent(Graphics g0) {
            Graphics2D g = (Graphics2D) g0.create();
            g.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
            g.setColor(TAN); g.fillOval(0, 4, 40, 40);
            g.setColor(DARK);
            g.setStroke(new BasicStroke(2.6f, BasicStroke.CAP_ROUND, BasicStroke.JOIN_ROUND));
            if (cart) {
                java.awt.geom.Path2D.Double p = new java.awt.geom.Path2D.Double();
                p.moveTo(9, 15); p.lineTo(13, 15); p.lineTo(16, 28); p.lineTo(29, 28); p.lineTo(31, 19); p.lineTo(14, 19);
                g.draw(p);
                g.fillOval(16, 31, 5, 5); g.fillOval(26, 31, 5, 5);
            } else {
                g.drawOval(11, 14, 15, 15);
                g.drawLine(24, 27, 30, 33);
            }
            if (cart && badge > 0) {
                g.setColor(RED); g.fillOval(26, 0, 20, 20);
                g.setColor(Color.WHITE); g.setFont(new Font("SansSerif", Font.BOLD, 11));
                String t = badge > 99 ? "99+" : String.valueOf(badge);
                FontMetrics fm = g.getFontMetrics();
                g.drawString(t, 36 - fm.stringWidth(t) / 2, 14);
            }
            g.dispose();
        }
    }

    JPanel sidebar(boolean mgr, String active) {
        JPanel s = new JPanel();
        s.setLayout(new BoxLayout(s, BoxLayout.Y_AXIS));
        s.setBackground(DARK); s.setPreferredSize(new Dimension(140, 0));
        List<String> items = new ArrayList<>(Arrays.asList(CATS));
        if (mgr) items.add("Sales Report");
        for (String c : items) {
            JButton b = btn(c, c.equals(active) ? TAN : DARK, Color.WHITE);
            b.setMaximumSize(new Dimension(Integer.MAX_VALUE, 42));
            b.setHorizontalAlignment(SwingConstants.LEFT);
            b.addActionListener(e -> {
                if (c.equals("Sales Report")) showSales();
                else {
                    Dashboard d = mgr ? mgrDash : empDash;
                    d.cat = c; d.refresh(); goTo(mgr ? "manager" : "employee");
                }
            });
            s.add(b);
        }
        return s;
    }

    void updateCartBtn() {
        int n = 0; for (int q : cart.values()) n += q;
        if (empDash != null && empDash.cartBtn != null) empDash.cartBtn.setBadge(n);
    }
    // Employee / Manager dashboard
    class Dashboard extends JPanel {
        final boolean mgr;
        String cat = "Coffee";
        final JPanel grid = new JPanel(new GridLayout(0, 3, 18, 18));
        final JLabel title = titleLabel("");
        JPanel side;
        IconButton cartBtn;

        Dashboard(boolean m) {
            mgr = m;
            setLayout(new BorderLayout());
            JPanel right = new JPanel(new FlowLayout(FlowLayout.RIGHT, 8, 0)); right.setOpaque(false);
            if (!mgr) {
                IconButton search = new IconButton(false);
                search.addActionListener(e -> goTo("track"));
                cartBtn = new IconButton(true);
                cartBtn.addActionListener(e -> showOrder());
                right.add(search); right.add(cartBtn);
            }
            right.add(new Avatar());
            JPanel h = new JPanel(new BorderLayout());
            h.setBackground(DARK); h.setBorder(new EmptyBorder(10, 20, 10, 20));
            title.setHorizontalAlignment(SwingConstants.CENTER);
            h.add(title, BorderLayout.CENTER); h.add(right, BorderLayout.EAST);
            JPanel pad = new JPanel(); pad.setOpaque(false); pad.setPreferredSize(new Dimension(230, 1));
            h.add(pad, BorderLayout.WEST);
            add(h, BorderLayout.NORTH);

            grid.setBorder(new EmptyBorder(20, 20, 20, 20));
            JPanel wrap = new JPanel(new BorderLayout()); wrap.setBackground(Color.WHITE);
            wrap.add(grid, BorderLayout.NORTH);
            grid.setBackground(Color.WHITE);
            JScrollPane sp = new JScrollPane(wrap); sp.setBorder(null);
            sp.getVerticalScrollBar().setUnitIncrement(16);
            add(sp, BorderLayout.CENTER);
            refresh();
        }

        void refresh() {
            title.setText(mgr ? "Manager" : employeeName);
            if (side != null) remove(side);
            side = sidebar(mgr, cat);
            add(side, BorderLayout.WEST);
            grid.removeAll();
            for (Product p : products) if (p.cat.equals(cat)) grid.add(mgr ? mgrCard(p) : empCard(p));
            if (mgr) grid.add(addCard());
            int fill = 3 - (grid.getComponentCount() % 3);
            if (fill < 3) for (int i = 0; i < fill; i++) { JPanel e = new JPanel(); e.setOpaque(false); grid.add(e); }
            revalidate(); repaint();
            updateCartBtn();
        }

        JPanel baseCard(Product p) {
            JPanel c = new JPanel(new BorderLayout(0, 4));
            c.setBackground(CARD); c.setBorder(BorderFactory.createCompoundBorder(
                    BorderFactory.createLineBorder(new Color(0xBBBBBB)), new EmptyBorder(8, 8, 8, 8)));
            c.setPreferredSize(new Dimension(180, 190));
            JLabel pic = new JLabel(p.img != null ? scale(p.img, 120, 85) : null, SwingConstants.CENTER);
            if (p.img == null) pic.setText(p.name);
            pic.setBackground(Color.WHITE); pic.setOpaque(true);
            c.add(pic, BorderLayout.CENTER);
            return c;
        }

        JPanel empCard(Product p) {
            JPanel c = baseCard(p);
            JLabel price = new JLabel(p.name + "  -  " + money(p.price), SwingConstants.CENTER);
            JButton minus = btn("-", Color.LIGHT_GRAY, Color.BLACK), plus = btn("+", TAN, Color.BLACK);
            minus.setBorder(new EmptyBorder(2, 10, 2, 10)); plus.setBorder(new EmptyBorder(2, 10, 2, 10));
            JLabel qty = new JLabel(String.valueOf(cart.getOrDefault(p, 0)), SwingConstants.CENTER);
            qty.setFont(BOLD); qty.setForeground(RED); qty.setPreferredSize(new Dimension(30, 20));
            minus.addActionListener(e -> {
                int q = cart.getOrDefault(p, 0) - 1;
                if (q <= 0) cart.remove(p); else cart.put(p, q);
                qty.setText(String.valueOf(Math.max(q, 0))); updateCartBtn();
            });
            plus.addActionListener(e -> {
                int q = cart.getOrDefault(p, 0) + 1; cart.put(p, q);
                qty.setText(String.valueOf(q)); updateCartBtn();
            });
            JPanel ctr = new JPanel(new FlowLayout(FlowLayout.CENTER, 6, 0)); ctr.setOpaque(false);
            ctr.add(minus); ctr.add(qty); ctr.add(plus);
            JPanel south = new JPanel(new GridLayout(2, 1, 0, 4)); south.setOpaque(false);
            south.add(price); south.add(ctr);
            c.add(south, BorderLayout.SOUTH);
            return c;
        }

        JPanel mgrCard(Product p) {
            JPanel c = baseCard(p);
            JButton edit = btn("Edit", TAN, Color.BLACK), del = btn("X", RED, Color.WHITE);
            edit.setBorder(new EmptyBorder(2, 10, 2, 10)); del.setBorder(new EmptyBorder(2, 10, 2, 10));
            edit.addActionListener(e -> editProduct(p, cat));
            del.addActionListener(e -> {
                if (JOptionPane.showConfirmDialog(CoffeeShopPOS.this, "Delete \"" + p.name + "\"?", "Delete product",
                        JOptionPane.YES_NO_OPTION) == JOptionPane.YES_OPTION) {
                    products.remove(p); cart.remove(p); empDash.refresh(); mgrDash.refresh();
                }
            });
            JPanel top = new JPanel(new BorderLayout()); top.setOpaque(false);
            JPanel tr = new JPanel(new FlowLayout(FlowLayout.RIGHT, 4, 0)); tr.setOpaque(false);
            tr.add(edit); tr.add(del); top.add(tr, BorderLayout.EAST);
            c.add(top, BorderLayout.NORTH);
            c.add(new JLabel(p.name + "  -  " + money(p.price), SwingConstants.CENTER), BorderLayout.SOUTH);
            return c;
        }

        JPanel addCard() {
            JPanel c = new JPanel(new BorderLayout());
            c.setBackground(CARD); c.setBorder(BorderFactory.createLineBorder(new Color(0xBBBBBB)));
            JButton plus = btn("+", CARD, Color.BLACK);
            plus.setFont(new Font("SansSerif", Font.BOLD, 40));
            plus.addActionListener(e -> editProduct(null, cat));
            c.add(plus); return c;
        }
    }
    // Add / edit product dialog
    void editProduct(Product existing, String cat) {
        JDialog d = new JDialog(this, existing == null ? "Add product" : "Edit product", true);
        JTextField name = new JTextField(existing == null ? "" : existing.name, 16);
        JTextField price = new JTextField(existing == null ? "" : String.valueOf(existing.price), 8);
        ImageIcon[] img = {existing == null ? null : existing.img};

        JButton imgBtn = btn("+   Add Image", new Color(0xC8A383), Color.BLACK);
        imgBtn.setPreferredSize(new Dimension(300, 140));
        if (img[0] != null) { imgBtn.setText(""); imgBtn.setIcon(scale(img[0], 280, 130)); }
        imgBtn.addActionListener(e -> {
            JFileChooser fc = new JFileChooser();
            fc.setFileFilter(new javax.swing.filechooser.FileNameExtensionFilter("Images", "png", "jpg", "jpeg", "gif"));
            if (fc.showOpenDialog(d) == JFileChooser.APPROVE_OPTION) {
                img[0] = new ImageIcon(fc.getSelectedFile().getAbsolutePath());
                imgBtn.setText(""); imgBtn.setIcon(scale(img[0], 280, 130));
            }
        });

        JButton cancel = btn("Cancel", RED, Color.WHITE), save = btn("Save", GREEN, Color.WHITE);
        cancel.addActionListener(e -> d.dispose());
        save.addActionListener(e -> {
            String n = name.getText().trim(); double p;
            if (n.isEmpty()) { JOptionPane.showMessageDialog(d, "Please enter a product name."); return; }
            try { p = Double.parseDouble(price.getText().trim()); if (p <= 0) throw new NumberFormatException(); }
            catch (NumberFormatException ex) { JOptionPane.showMessageDialog(d, "Please enter a valid price."); return; }
            if (existing == null) { Product np = new Product(n, p, cat); np.img = img[0]; products.add(np); }
            else { existing.name = n; existing.price = p; existing.img = img[0]; }
            empDash.refresh(); mgrDash.refresh(); d.dispose();
        });

        JPanel form = new JPanel(); form.setLayout(new BoxLayout(form, BoxLayout.Y_AXIS));
        form.setBorder(new EmptyBorder(20, 20, 20, 20)); form.setBackground(Color.WHITE);
        JPanel r0 = new JPanel(); r0.setOpaque(false); r0.add(imgBtn);
        JPanel r1 = new JPanel(new FlowLayout(FlowLayout.LEFT)); r1.setOpaque(false); r1.add(new JLabel("Product Name:")); r1.add(name);
        JPanel r2 = new JPanel(new FlowLayout(FlowLayout.LEFT)); r2.setOpaque(false); r2.add(new JLabel("Price:")); r2.add(price);
        JPanel r3 = new JPanel(new GridLayout(1, 2, 20, 0)); r3.setOpaque(false); r3.add(cancel); r3.add(save);
        form.add(r0); form.add(r1); form.add(r2); form.add(Box.createVerticalStrut(10)); form.add(r3);
        d.setContentPane(form); d.pack(); d.setLocationRelativeTo(this); d.setVisible(true);
    }
    // Order Management (cart + customer preferences)
    JPanel radios(String label, ButtonGroup g, Runnable onChange, String... opts) {
        JPanel p = new JPanel(new FlowLayout(FlowLayout.LEFT, 14, 2)); p.setOpaque(false);
        JLabel l = new JLabel(label); l.setFont(BOLD); p.add(l);
        boolean first = true;
        for (String o : opts) {
            JRadioButton r = new JRadioButton(o, first); first = false;
            r.setOpaque(false); r.setActionCommand(o);
            r.addActionListener(e -> onChange.run());
            g.add(r); p.add(r);
        }
        return p;
    }

    void showOrder() {
        orderPanel.removeAll(); orderPanel.setLayout(new BorderLayout());
        JButton back = btn("Back", TAN, Color.BLACK);
        back.addActionListener(e -> goTo("employee"));
        IconButton cartIcon = new IconButton(true);
        int count = 0; for (int q : cart.values()) count += q;
        cartIcon.setBadge(count); cartIcon.setToolTipText("Back to menu");
        cartIcon.addActionListener(e -> goTo("employee"));
        JPanel hr = new JPanel(new FlowLayout(FlowLayout.RIGHT, 10, 0)); hr.setOpaque(false);
        hr.add(back); hr.add(cartIcon);
        orderPanel.add(header(titleLabel("Order Management"), false, hr), BorderLayout.NORTH);

        Runnable[] up = new Runnable[1];
        Runnable change = () -> up[0].run();
        ButtonGroup gSize = new ButtonGroup(), gSugar = new ButtonGroup(), gPay = new ButtonGroup(),
                gType = new ButtonGroup(), gDisc = new ButtonGroup();

        JPanel rows = new JPanel(new GridLayout(0, 5, 8, 8)); rows.setOpaque(false);
        JLabel items = new JLabel(), total = new JLabel();
        items.setFont(BOLD); total.setFont(new Font("SansSerif", Font.BOLD, 16));

        JPanel body = new JPanel(); body.setLayout(new BoxLayout(body, BoxLayout.Y_AXIS));
        body.setBackground(BG); body.setBorder(new EmptyBorder(15, 25, 15, 25));
        JLabel ot = new JLabel("Orders"); ot.setFont(new Font("SansSerif", Font.BOLD, 18));
        body.add(ot); body.add(Box.createVerticalStrut(6)); body.add(items);
        body.add(Box.createVerticalStrut(8)); body.add(rows);
        body.add(Box.createVerticalStrut(8)); body.add(total);
        body.add(Box.createVerticalStrut(16));
        JLabel cp = new JLabel("Customer Preference"); cp.setFont(new Font("SansSerif", Font.BOLD, 18)); body.add(cp);
        body.add(radios("Sizes:", gSize, change, SIZE_ADD.keySet().toArray(new String[0])));
        body.add(radios("Sugar Level:", gSugar, change, "0%", "25%", "50%", "75%"));
        body.add(radios("Mode of Payment:", gPay, change, "Cash", "E-Wallet"));
        body.add(radios("Order Type:", gType, change, "Dine-In", "Takeout"));
        body.add(radios("Discounts:", gDisc, change, DISC.keySet().toArray(new String[0])));

        up[0] = () -> {
            rows.removeAll();
            for (String h : new String[]{"Description", "Qty", "Price", "Total", "Remove"}) {
                JLabel l = new JLabel(h); l.setFont(BOLD); rows.add(l);
            }
            double sub = 0; int n = 0;
            for (Map.Entry<Product, Integer> en : new ArrayList<>(cart.entrySet())) {
                Product p = en.getKey(); int q = en.getValue(); double u = unit(p, sel(gSize));
                rows.add(new JLabel(p.name)); rows.add(new JLabel(String.valueOf(q)));
                rows.add(new JLabel(money(u))); rows.add(new JLabel(money(u * q)));
                JButton rm = btn("X", RED, Color.WHITE); rm.setBorder(new EmptyBorder(2, 8, 2, 8));
                rm.addActionListener(e -> { cart.remove(p); updateCartBtn(); up[0].run(); });
                JPanel w = new JPanel(new FlowLayout(FlowLayout.LEFT, 0, 0)); w.setOpaque(false); w.add(rm); rows.add(w);
                sub += u * q; n += q;
            }
            double disc = sub * DISC.get(sel(gDisc));
            items.setText("No. of items: " + n);
            total.setText("Subtotal " + money(sub) + "   |   Discount -" + money(disc) + "   |   Total " + money(sub - disc));
            rows.revalidate(); rows.repaint();
        };
        up[0].run();

        JScrollPane sp = new JScrollPane(body); sp.setBorder(null);
        orderPanel.add(sp, BorderLayout.CENTER);

        JButton cancel = btn("Cancel", RED, Color.WHITE), proceed = btn("Proceed", GREEN, Color.WHITE);
        cancel.addActionListener(e -> {
            if (cart.isEmpty() || JOptionPane.showConfirmDialog(this, "Cancel this order and clear the cart?",
                    "Cancel order", JOptionPane.YES_NO_OPTION) == JOptionPane.YES_OPTION) {
                cart.clear(); updateCartBtn(); empDash.refresh(); goTo("employee");
            }
        });
        proceed.addActionListener(e -> {
            if (cart.isEmpty()) { JOptionPane.showMessageDialog(this, "The cart is empty. Add items first."); return; }
            Order o = new Order();
            o.size = sel(gSize); o.sugar = sel(gSugar); o.payment = sel(gPay); o.type = sel(gType); o.discName = sel(gDisc);
            for (Map.Entry<Product, Integer> en : cart.entrySet()) {
                Product p = en.getKey(); double u = unit(p, o.size);
                String label = p.cat.equals("Food") ? p.name : p.name + " (" + o.size + ")";
                o.lines.add(new Line(label, p.name, en.getValue(), u));
                o.subtotal += u * en.getValue();
            }
            o.discount = o.subtotal * DISC.get(o.discName);
            o.total = o.subtotal - o.discount;
            o.paid = o.total;
            if (o.payment.equals("Cash")) {
                String s = JOptionPane.showInputDialog(this, "Total: " + money(o.total) + "\nAmount paid (PHP):");
                if (s == null) return;
                try { o.paid = Double.parseDouble(s.trim()); }
                catch (NumberFormatException ex) { JOptionPane.showMessageDialog(this, "Enter a valid amount."); return; }
                if (o.paid < o.total) { JOptionPane.showMessageDialog(this, "Amount paid is less than the total."); return; }
            }
            do { o.code = "CEV-" + (100000 + new Random().nextInt(900000)); } while (orders.containsKey(o.code));
            orders.put(o.code, o);
            cart.clear(); updateCartBtn(); empDash.refresh();
            showReceipt(o, "employee");
        });
        JPanel south = new JPanel(new GridLayout(1, 2, 20, 0));
        south.setBackground(BG); south.setBorder(new EmptyBorder(10, 25, 15, 25));
        south.add(cancel); south.add(proceed);
        orderPanel.add(south, BorderLayout.SOUTH);

        orderPanel.revalidate(); orderPanel.repaint();
        goTo("order");
    }
    // Receipt
    String receiptText(Order o) {
        DateTimeFormatter df = DateTimeFormatter.ofPattern("MMMM dd, yyyy"), tf = DateTimeFormatter.ofPattern("hh:mm a");
        StringBuilder sb = new StringBuilder();
        String line = "------------------------------------------\n";
        sb.append("             Coffeum Est Vita\n").append(line);
        sb.append(String.format("Tracking Code    : %s%n", o.code));
        sb.append(String.format("Transaction Date : %s%n", o.time.format(df)));
        sb.append(String.format("Transaction Time : %s%n", o.time.format(tf)));
        sb.append(String.format("Order Type       : %s%n", o.type));
        sb.append(String.format("Sugar Level      : %s%n", o.sugar)).append(line);
        sb.append(String.format("%-22s %3s %13s%n", "Purchased Products", "Qty", "Price"));
        for (Line l : o.lines)
            sb.append(String.format("%-22s %3d %13s%n", l.label.length() > 22 ? l.label.substring(0, 22) : l.label, l.qty, money(l.unit * l.qty)));
        sb.append(line);
        sb.append(String.format("%-26s %15s%n", "Subtotal", money(o.subtotal)));
        sb.append(String.format("%-26s %15s%n", "Discount (" + o.discName + ")", "-" + money(o.discount)));
        sb.append(String.format("%-26s %15s%n", "Total Amount", money(o.total))).append(line);
        sb.append(String.format("%-26s %15s%n", "Payment Method", o.payment));
        sb.append(String.format("%-26s %15s%n", "Amount Paid", money(o.paid)));
        sb.append(String.format("%-26s %15s%n", "Change", money(o.paid - o.total))).append(line);
        sb.append("               Thank You!\n");
        return sb.toString();
    }

    void showReceipt(Order o, String backCard) {
        receiptPanel.removeAll(); receiptPanel.setLayout(new BorderLayout());
        receiptPanel.add(header(titleLabel("Receipt Generation"), false, null), BorderLayout.NORTH);
        JTextArea area = new JTextArea(receiptText(o));
        area.setEditable(false); area.setFont(new Font(Font.MONOSPACED, Font.PLAIN, 14));
        area.setBorder(new EmptyBorder(20, 30, 20, 30));
        JPanel center = new JPanel(new GridBagLayout()); center.setBackground(BG);
        JScrollPane sp = new JScrollPane(area); sp.setPreferredSize(new Dimension(520, 500));
        center.add(sp);
        receiptPanel.add(center, BorderLayout.CENTER);

        JButton print = btn("Print Receipt", DARK, Color.WHITE), done = btn("Done", TAN, Color.BLACK);
        print.addActionListener(e -> {
            try { area.print(); }
            catch (Exception ex) { JOptionPane.showMessageDialog(this, "Could not print: " + ex.getMessage()); }
        });
        done.addActionListener(e -> goTo(backCard));
        JPanel south = new JPanel(new FlowLayout(FlowLayout.CENTER, 20, 10)); south.setBackground(BG);
        south.add(print); south.add(done);
        receiptPanel.add(south, BorderLayout.SOUTH);
        receiptPanel.revalidate(); receiptPanel.repaint();
        goTo("receipt");
    }
    // Order Tracking
    void buildTrack() {
        trackPanel = new JPanel(new BorderLayout());
        JButton back = btn("Back", TAN, Color.BLACK);
        back.addActionListener(e -> goTo("employee"));
        IconButton searchIcon = new IconButton(false);
        searchIcon.setToolTipText("Back to menu");
        searchIcon.addActionListener(e -> goTo("employee"));
        JPanel hr = new JPanel(new FlowLayout(FlowLayout.RIGHT, 10, 0)); hr.setOpaque(false);
        hr.add(back); hr.add(searchIcon);
        trackPanel.add(header(titleLabel("Order Tracking"), false, hr), BorderLayout.NORTH);

        JTextField code = new JTextField(14); code.setFont(new Font("SansSerif", Font.PLAIN, 18));
        code.setHorizontalAlignment(SwingConstants.CENTER);
        JButton go = btn("Search", DARK, Color.WHITE);
        Runnable search = () -> {
            Order o = orders.get(code.getText().trim().toUpperCase());
            if (o == null) JOptionPane.showMessageDialog(this, "No order found with that tracking code.");
            else { showReceipt(o, "track"); code.setText(""); }
        };
        go.addActionListener(e -> search.run());
        code.addActionListener(e -> search.run());

        JPanel box = new JPanel(); box.setLayout(new BoxLayout(box, BoxLayout.Y_AXIS));
        box.setBackground(BG); box.setBorder(new EmptyBorder(60, 120, 60, 120));
        JLabel l = new JLabel("Enter Unique Tracking Code:"); l.setFont(new Font("SansSerif", Font.PLAIN, 22));
        l.setAlignmentX(CENTER_ALIGNMENT); code.setAlignmentX(CENTER_ALIGNMENT); go.setAlignmentX(CENTER_ALIGNMENT);
        code.setMaximumSize(new Dimension(300, 40));
        box.add(l); box.add(Box.createVerticalStrut(15)); box.add(code);
        box.add(Box.createVerticalStrut(15)); box.add(go);
        JPanel center = new JPanel(new GridBagLayout()); center.setBackground(Color.WHITE); center.add(box);
        trackPanel.add(center, BorderLayout.CENTER);
    }
    // Sales report (manager)
    static boolean inPeriod(LocalDateTime t, String p) {
        LocalDate d = t.toLocalDate(), n = LocalDate.now();
        switch (p) {
            case "Daily": return d.equals(n);
            case "Weekly": return !d.isBefore(n.minusDays(6));
            case "Monthly": return d.getYear() == n.getYear() && d.getMonth() == n.getMonth();
            default: return d.getYear() == n.getYear();
        }
    }

    void showSales() {
        salesHolder.removeAll(); salesHolder.setLayout(new BorderLayout());
        salesHolder.add(header(titleLabel("Manager"), true, new Avatar()), BorderLayout.NORTH);
        salesHolder.add(sidebar(true, "Sales Report"), BorderLayout.WEST);

        DefaultTableModel m = new DefaultTableModel(new String[]{"Product", "Quantity", "Total Sales"}, 0) {
            @Override public boolean isCellEditable(int r, int c) { return false; }
        };
        JTable table = new JTable(m); table.setRowHeight(26);
        JLabel summary = new JLabel(), net = new JLabel();
        summary.setFont(BOLD); net.setFont(new Font("SansSerif", Font.BOLD, 18));
        String[] cur = {"Daily"};
        Runnable[] load = new Runnable[1];
        load[0] = () -> {
            m.setRowCount(0);
            Map<String, double[]> agg = new TreeMap<>();
            double netSales = 0; int count = 0;
            for (Order o : orders.values()) {
                if (!inPeriod(o.time, cur[0])) continue;
                count++; netSales += o.total;
                for (Line l : o.lines) {
                    double[] a = agg.computeIfAbsent(l.base, k -> new double[2]);
                    a[0] += l.qty; a[1] += l.qty * l.unit;
                }
            }
            agg.forEach((k, a) -> m.addRow(new Object[]{k, (int) a[0], money(a[1])}));
            summary.setText("Sales Summary - Orders: " + count);
            net.setText("Net Sales: " + money(netSales));
        };

        JPanel top = new JPanel(new FlowLayout(FlowLayout.LEFT, 8, 0)); top.setOpaque(false);
        JLabel rp = new JLabel("Report Period:"); rp.setFont(BOLD); top.add(rp);
        ButtonGroup g = new ButtonGroup();
        for (String p : new String[]{"Daily", "Weekly", "Monthly", "Annually"}) {
            JToggleButton b = new JToggleButton(p, p.equals("Daily"));
            b.setFocusPainted(false); b.setBackground(TAN); b.setOpaque(true);
            g.add(b); top.add(b);
            b.addActionListener(e -> { cur[0] = p; load[0].run(); });
        }
        JPanel c = new JPanel(new BorderLayout(0, 12)); c.setBackground(Color.WHITE); c.setBorder(new EmptyBorder(20, 20, 20, 20));
        c.add(top, BorderLayout.NORTH);
        c.add(new JScrollPane(table), BorderLayout.CENTER);
        JPanel bottom = new JPanel(new BorderLayout()); bottom.setOpaque(false);
        bottom.add(summary, BorderLayout.WEST); bottom.add(net, BorderLayout.EAST);
        c.add(bottom, BorderLayout.SOUTH);
        salesHolder.add(c, BorderLayout.CENTER);
        load[0].run();
        salesHolder.revalidate(); salesHolder.repaint();
        goTo("sales");
    }
}
