import java.awt.*;
import java.awt.event.*;
import java.util.*;
import javax.swing.*;
import javax.swing.table.*;

// -------------------- CRYPTO CLASS --------------------
class Crypto {
    private String name;
    private String symbol;
    private double priceUSD;

    public Crypto(String name, String symbol, double priceUSD){
        this.name = name;
        this.symbol = symbol;
        this.priceUSD = priceUSD;
    }

    public String getName(){ return name; }
    public String getSymbol(){ return symbol; }
    public double getPriceUSD(){ return priceUSD; }
    public void setPriceUSD(double price){ this.priceUSD = price; }
}

// -------------------- INVESTOR CLASS --------------------
class Investor {
    private String username;
    private String password;
    private HashMap<String, Double> portfolio = new HashMap<>();

    public Investor(String username, String password){
        this.username = username;
        this.password = password;
    }

    public String getUsername(){ return username; }
    public void setUsername(String username){ this.username = username; }
    public boolean checkPassword(String pw){ return password.equals(pw); }
    public void setPassword(String pw){ this.password = pw; }

    public HashMap<String, Double> getPortfolio(){ return portfolio; }

    public void buyCrypto(Crypto c, double qty){
        portfolio.put(c.getName(), portfolio.getOrDefault(c.getName(),0.0)+qty);
    }

    public boolean sellCrypto(Crypto c, double qty){
        double owned = portfolio.getOrDefault(c.getName(),0.0);
        if(qty > owned) return false;
        portfolio.put(c.getName(), owned-qty);
        return true;
    }
}

// -------------------- MAIN GUI --------------------
public class MainGui {
    private ArrayList<Crypto> cryptos = new ArrayList<>();
    private ArrayList<Investor> investors = new ArrayList<>();
    private Investor currentInvestor = null;

    private JFrame frame;
    private JTable table;
    private DefaultTableModel tableModel;
    private JLabel lblPortfolio;
    private JComboBox<String> currencyBox;
    private JLabel lblUsernameTop;

    private HashMap<String, Double> currencyRates = new HashMap<>();

    // accent colors
    private final Color BG_DARK = new Color(18, 18, 24);
    private final Color BG_PANEL = new Color(28, 28, 36);
    private final Color ACCENT = new Color(0, 195, 255);
    private final Color ACCENT_SOFT = new Color(0, 120, 200);

    public MainGui(){
        initGlobalUI();

        // Sample cryptos (10 coins)
        cryptos.add(new Crypto("Bitcoin","BTC",30000.0));
        cryptos.add(new Crypto("Ethereum","ETH",2000.0));
        cryptos.add(new Crypto("Binance Coin","BNB",350.0));
        cryptos.add(new Crypto("Cardano","ADA",1.2));
        cryptos.add(new Crypto("Dogecoin","DOGE",0.08));
        cryptos.add(new Crypto("XRP","XRP",0.7));
        cryptos.add(new Crypto("Solana","SOL",100.0));
        cryptos.add(new Crypto("Polkadot","DOT",25.0));
        cryptos.add(new Crypto("Litecoin","LTC",90.0));
        cryptos.add(new Crypto("Chainlink","LINK",15.0));

        // Currency rates
        currencyRates.put("USD",1.0);
        currencyRates.put("PKR",280.0);
        currencyRates.put("EUR",0.93);
        currencyRates.put("GBP",0.81);
        currencyRates.put("JPY",146.5);

        showLogin();
    }

    // ---------------- GLOBAL UI TWEAKS ----------------
    private void initGlobalUI() {
        UIManager.put("Button.focusPainted", Boolean.FALSE);
        UIManager.put("Table.gridColor", new Color(55,55,65));
        UIManager.put("TableHeader.background", new Color(30,30,40));
        UIManager.put("TableHeader.foreground", Color.WHITE);
        UIManager.put("TableHeader.font", new Font("Segoe UI", Font.BOLD, 13));
        UIManager.put("OptionPane.background", BG_PANEL);
        UIManager.put("Panel.background", BG_PANEL);
    }

    private void styleFlatButton(JButton btn, Color bg, Color fg) {
        btn.setBackground(bg);
        btn.setForeground(fg);
        btn.setFocusPainted(false);
        btn.setBorderPainted(false);
        btn.setFont(new Font("Segoe UI", Font.BOLD, 13));
        btn.setCursor(Cursor.getPredefinedCursor(Cursor.HAND_CURSOR));
    }

    // ---------------- LOGIN / SIGNUP ----------------
    private void showLogin(){
        JFrame loginFrame = new JFrame("Crypto Portfolio Login");
        loginFrame.setSize(420,320);
        loginFrame.setLocationRelativeTo(null);
        loginFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        loginFrame.getContentPane().setBackground(BG_DARK);
        loginFrame.setLayout(null);

        JLabel title = new JLabel("Crypto Portfolio Tracker");
        title.setForeground(ACCENT);
        title.setFont(new Font("Segoe UI", Font.BOLD, 18));
        title.setBounds(80,15,300,30);
        loginFrame.add(title);

        JLabel lblUser = new JLabel("Username");
        lblUser.setForeground(Color.LIGHT_GRAY);
        lblUser.setBounds(60,70,80,25);
        loginFrame.add(lblUser);

        JTextField tfUser = new JTextField();
        tfUser.setBounds(150,70,180,25);
        loginFrame.add(tfUser);

        JLabel lblPass = new JLabel("Password");
        lblPass.setForeground(Color.LIGHT_GRAY);
        lblPass.setBounds(60,110,80,25);
        loginFrame.add(lblPass);

        JPasswordField pfPass = new JPasswordField();
        pfPass.setBounds(150,110,180,25);
        loginFrame.add(pfPass);

        JButton btnLogin = new JButton("Login");
        styleFlatButton(btnLogin, ACCENT, Color.BLACK);
        btnLogin.setBounds(60,170,120,35);
        loginFrame.add(btnLogin);

        JButton btnSignup = new JButton("Sign up");
        styleFlatButton(btnSignup, BG_PANEL, Color.WHITE);
        btnSignup.setBounds(210,170,120,35);
        loginFrame.add(btnSignup);

        JLabel hint = new JLabel("Admin: admin / admin123");
        hint.setForeground(Color.GRAY);
        hint.setFont(new Font("Segoe UI", Font.PLAIN, 11));
        hint.setBounds(120,220,200,20);
        loginFrame.add(hint);

        String adminUser = "admin";
        String adminPass = "admin123";

        btnLogin.addActionListener(e -> {
            String user = tfUser.getText().trim();
            String pass = new String(pfPass.getPassword()).trim();

            if(user.equals(adminUser) && pass.equals(adminPass)){
                JOptionPane.showMessageDialog(loginFrame,"Welcome Admin!");
                showAdminPanel();
                loginFrame.dispose();
                return;
            }

            for(Investor inv: investors){
                if(inv.getUsername().equals(user) && inv.checkPassword(pass)){
                    currentInvestor = inv;
                    JOptionPane.showMessageDialog(loginFrame,"Login Success!");
                    loginFrame.dispose();
                    showInvestorPanel();
                    return;
                }
            }
            JOptionPane.showMessageDialog(loginFrame,"Invalid credentials!");
        });

        btnSignup.addActionListener(e -> {
            String user = tfUser.getText().trim();
            String pass = new String(pfPass.getPassword()).trim();
            if(user.isEmpty() || pass.isEmpty()){
                JOptionPane.showMessageDialog(loginFrame,"Enter username and password!");
                return;
            }
            for(Investor inv: investors){
                if(inv.getUsername().equals(user)){
                    JOptionPane.showMessageDialog(loginFrame,"Username already taken!");
                    return;
                }
            }
            investors.add(new Investor(user, pass));
            JOptionPane.showMessageDialog(loginFrame,"Account created! Please login.");
        });

        loginFrame.setVisible(true);
    }

    // ---------------- INVESTOR PANEL ----------------
    private void showInvestorPanel(){
        frame = new JFrame("Crypto Dashboard");
        frame.setSize(1000,620);
        frame.setLocationRelativeTo(null);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.getContentPane().setBackground(BG_DARK);
        frame.setLayout(new BorderLayout(10,10));

        // TOP BAR
        JPanel topBar = new JPanel(new BorderLayout());
        topBar.setBackground(BG_PANEL);
        topBar.setBorder(BorderFactory.createEmptyBorder(10,15,10,15));

        JLabel appTitle = new JLabel("Crypto Portfolio Dashboard");
        appTitle.setForeground(ACCENT);
        appTitle.setFont(new Font("Segoe UI", Font.BOLD, 18));

        lblUsernameTop = new JLabel("Logged in as: " + currentInvestor.getUsername());
        lblUsernameTop.setForeground(Color.LIGHT_GRAY);
        lblUsernameTop.setFont(new Font("Segoe UI", Font.PLAIN, 13));
        lblUsernameTop.setHorizontalAlignment(SwingConstants.RIGHT);

        topBar.add(appTitle, BorderLayout.WEST);
        topBar.add(lblUsernameTop, BorderLayout.EAST);

        frame.add(topBar, BorderLayout.NORTH);

        // SIDEBAR (LEFT)
        JPanel sidebar = new JPanel();
        sidebar.setBackground(BG_PANEL);
        sidebar.setPreferredSize(new Dimension(200, frame.getHeight()));
        sidebar.setLayout(new BoxLayout(sidebar, BoxLayout.Y_AXIS));
        sidebar.setBorder(BorderFactory.createEmptyBorder(15,10,15,10));

        JLabel navTitle = new JLabel("MENU");
        navTitle.setForeground(Color.GRAY);
        navTitle.setFont(new Font("Segoe UI", Font.BOLD, 14));
        navTitle.setAlignmentX(Component.LEFT_ALIGNMENT);
        sidebar.add(navTitle);
        sidebar.add(Box.createRigidArea(new Dimension(0,10)));

        JButton btnSettings = new JButton("Account Settings");
        styleFlatButton(btnSettings, new Color(45,45,55), Color.WHITE);
        btnSettings.setAlignmentX(Component.LEFT_ALIGNMENT);
        sidebar.add(btnSettings);
        sidebar.add(Box.createRigidArea(new Dimension(0,8)));

        JButton btnLogout = new JButton("Logout");
        styleFlatButton(btnLogout, new Color(180,50,70), Color.WHITE);
        btnLogout.setAlignmentX(Component.LEFT_ALIGNMENT);
        sidebar.add(btnLogout);

        sidebar.add(Box.createVerticalGlue());

        frame.add(sidebar, BorderLayout.WEST);

        // CENTER PANEL (TABLE + SIDE SUMMARY)
        JPanel center = new JPanel(new BorderLayout(10,10));
        center.setBackground(BG_DARK);
        center.setBorder(BorderFactory.createEmptyBorder(10,10,10,10));

        // TABLE PANEL
        JPanel tablePanel = new JPanel(new BorderLayout());
        tablePanel.setBackground(BG_PANEL);
        tablePanel.setBorder(BorderFactory.createTitledBorder(
                BorderFactory.createLineBorder(new Color(60,60,80)),
                "Market & Holdings",
                0, 0,
                new Font("Segoe UI", Font.BOLD, 13),
                Color.LIGHT_GRAY
        ));

        String[] cols = {"Coin","Symbol","Price (USD)","Owned"};
        tableModel = new DefaultTableModel(cols,0){
            @Override
            public boolean isCellEditable(int row,int column){ return false; }
        };
        table = new JTable(tableModel);
        table.setRowHeight(28);
        table.setBackground(new Color(32,32,40));
        table.setForeground(Color.WHITE);
        table.setSelectionBackground(new Color(60,60,90));
        table.setFont(new Font("Segoe UI",Font.PLAIN,13));

        JTableHeader header = table.getTableHeader();
        header.setBackground(new Color(25,25,35));
        header.setForeground(Color.WHITE);
        header.setFont(new Font("Segoe UI", Font.BOLD, 13));
        header.setPreferredSize(new Dimension(header.getWidth(), 28));

        JScrollPane scroll = new JScrollPane(table);
        scroll.getViewport().setBackground(BG_PANEL);
        tablePanel.add(scroll, BorderLayout.CENTER);

        center.add(tablePanel, BorderLayout.CENTER);

        // RIGHT SUMMARY PANEL
        JPanel summaryPanel = new JPanel();
        summaryPanel.setBackground(BG_PANEL);
        summaryPanel.setPreferredSize(new Dimension(260, frame.getHeight()));
        summaryPanel.setLayout(new BoxLayout(summaryPanel, BoxLayout.Y_AXIS));
        summaryPanel.setBorder(BorderFactory.createEmptyBorder(15,15,15,15));

        JLabel lblSummaryTitle = new JLabel("Portfolio Summary");
        lblSummaryTitle.setForeground(Color.WHITE);
        lblSummaryTitle.setFont(new Font("Segoe UI", Font.BOLD, 15));
        lblSummaryTitle.setAlignmentX(Component.LEFT_ALIGNMENT);
        summaryPanel.add(lblSummaryTitle);
        summaryPanel.add(Box.createRigidArea(new Dimension(0,10)));

        lblPortfolio = new JLabel("Portfolio Value: $0.00");
        lblPortfolio.setForeground(ACCENT);
        lblPortfolio.setFont(new Font("Segoe UI", Font.BOLD, 16));
        lblPortfolio.setAlignmentX(Component.LEFT_ALIGNMENT);
        summaryPanel.add(lblPortfolio);
        summaryPanel.add(Box.createRigidArea(new Dimension(0,10)));

        JPanel currencyPanel = new JPanel(new FlowLayout(FlowLayout.LEFT, 5,5));
        currencyPanel.setBackground(BG_PANEL);
        JLabel lblCurr = new JLabel("View in:");
        lblCurr.setForeground(Color.LIGHT_GRAY);
        currencyBox = new JComboBox<>(new String[]{"USD","PKR","EUR","GBP","JPY"});
        currencyBox.setBackground(new Color(40,40,50));
        currencyBox.setForeground(Color.WHITE);
        currencyPanel.add(lblCurr);
        currencyPanel.add(currencyBox);
        currencyPanel.setAlignmentX(Component.LEFT_ALIGNMENT);
        summaryPanel.add(currencyPanel);

        summaryPanel.add(Box.createRigidArea(new Dimension(0,15)));

        JButton btnRefreshMarket = new JButton("Refresh Market Prices");
        styleFlatButton(btnRefreshMarket, ACCENT_SOFT, Color.WHITE);
        btnRefreshMarket.setAlignmentX(Component.LEFT_ALIGNMENT);
        summaryPanel.add(btnRefreshMarket);

        summaryPanel.add(Box.createRigidArea(new Dimension(0,15)));

        JLabel info = new JLabel("<html><body style='width:200px'>"
                + "Tip: Click any coin row to Buy or Sell holdings."
                + "</body></html>");
        info.setForeground(Color.GRAY);
        info.setFont(new Font("Segoe UI", Font.PLAIN, 12));
        info.setAlignmentX(Component.LEFT_ALIGNMENT);
        summaryPanel.add(info);

        center.add(summaryPanel, BorderLayout.EAST);

        frame.add(center, BorderLayout.CENTER);

        // UPDATE TABLE + LABEL
        updateTable();
        updatePortfolioLabel();

        currencyBox.addActionListener(e -> updatePortfolioLabel());

        // TABLE CLICK HANDLER
        table.addMouseListener(new MouseAdapter(){
            public void mouseClicked(MouseEvent e){
                int row = table.getSelectedRow();
                if(row<0) return;
                Crypto c = cryptos.get(row);
                String[] options = {"Buy","Sell","Cancel"};
                int choice = JOptionPane.showOptionDialog(frame,"Choose action for "+c.getName(),
                        "Trade",JOptionPane.DEFAULT_OPTION,JOptionPane.QUESTION_MESSAGE,
                        null,options,options[0]);
                if(choice==0){ // Buy
                    String qtyStr = JOptionPane.showInputDialog(frame,"Enter quantity to buy:");
                    try{
                        double qty = Double.parseDouble(qtyStr);
                        if(qty<=0){ JOptionPane.showMessageDialog(frame,"Quantity must be positive!"); return;}
                        currentInvestor.buyCrypto(c,qty);
                        JOptionPane.showMessageDialog(frame,"Bought "+qty+" "+c.getName());
                        updateTable();
                        updatePortfolioLabel();
                    }catch(Exception ex){ JOptionPane.showMessageDialog(frame,"Invalid input");}
                }else if(choice==1){ // Sell
                    String qtyStr = JOptionPane.showInputDialog(frame,"Enter quantity to sell:");
                    try{
                        double qty = Double.parseDouble(qtyStr);
                        if(qty<=0){ JOptionPane.showMessageDialog(frame,"Quantity must be positive!"); return;}
                        boolean success = currentInvestor.sellCrypto(c,qty);
                        if(!success){
                            JOptionPane.showMessageDialog(frame,"You only own "+currentInvestor.getPortfolio().getOrDefault(c.getName(),0.0)+" "+c.getName());
                        } else {
                            JOptionPane.showMessageDialog(frame,"Sold "+qty+" "+c.getName());
                        }
                        updateTable();
                        updatePortfolioLabel();
                    }catch(Exception ex){ JOptionPane.showMessageDialog(frame,"Invalid input");}
                }
            }
        });

        // SETTINGS
        btnSettings.addActionListener(e -> showSettings());

        // LOGOUT
        btnLogout.addActionListener(e -> {
            frame.dispose();
            currentInvestor = null;
            showLogin();
        });

        // REFRESH MARKET PRICES (REALISTIC FEEL)
        btnRefreshMarket.addActionListener(e -> {
            Random rnd = new Random();
            for(Crypto c : cryptos){
                double old = c.getPriceUSD();
                // random change between -5% to +5%
                double factor = 0.95 + (0.10 * rnd.nextDouble());
                double updated = old * factor;
                c.setPriceUSD(Math.max(0.01, updated));
            }
            updateTable();
            updatePortfolioLabel();
            JOptionPane.showMessageDialog(frame,"Market prices updated!");
        });

        frame.setVisible(true);
    }

    private void showSettings(){
        JFrame settings = new JFrame("Account Settings");
        settings.setSize(420,260);
        settings.setLocationRelativeTo(frame);
        settings.getContentPane().setBackground(BG_DARK);
        settings.setLayout(null);

        JLabel title = new JLabel("Account Settings");
        title.setForeground(ACCENT);
        title.setFont(new Font("Segoe UI", Font.BOLD, 16));
        title.setBounds(130,15,200,25);
        settings.add(title);

        JLabel lblUser = new JLabel("Username");
        lblUser.setForeground(Color.LIGHT_GRAY); lblUser.setBounds(40,60,80,25); settings.add(lblUser);
        JTextField tfUser = new JTextField(currentInvestor.getUsername()); tfUser.setBounds(140,60,220,25); settings.add(tfUser);

        JLabel lblPass = new JLabel("Password");
        lblPass.setForeground(Color.LIGHT_GRAY); lblPass.setBounds(40,100,80,25); settings.add(lblPass);
        JPasswordField pfPass = new JPasswordField(); pfPass.setBounds(140,100,220,25); settings.add(pfPass);

        JButton btnSave = new JButton("Save Changes");
        styleFlatButton(btnSave, ACCENT_SOFT, Color.WHITE);
        btnSave.setBounds(140,150,150,35); settings.add(btnSave);

        btnSave.addActionListener(e -> {
            String newUser = tfUser.getText().trim();
            String newPass = new String(pfPass.getPassword()).trim();
            if(!newUser.isEmpty()) currentInvestor.setUsername(newUser);
            if(!newPass.isEmpty()) currentInvestor.setPassword(newPass);
            lblUsernameTop.setText("Logged in as: " + currentInvestor.getUsername());
            JOptionPane.showMessageDialog(settings,"Settings updated!");
            settings.dispose();
        });

        settings.setVisible(true);
    }

    // ---------------- UPDATE TABLE & PORTFOLIO ----------------
    private void updateTable(){
        if(tableModel == null) return;
        tableModel.setRowCount(0);
        for(Crypto c: cryptos){
            double owned = currentInvestor.getPortfolio().getOrDefault(c.getName(),0.0);
            tableModel.addRow(new Object[]{c.getName(),c.getSymbol(),
                    String.format("%.2f",c.getPriceUSD()),owned});
        }
    }

    private void updatePortfolioLabel(){
        if(lblPortfolio == null || currencyBox == null) return;
        double total =0;
        String currency = (String)currencyBox.getSelectedItem();
        for(Crypto c: cryptos){
            double qty = currentInvestor.getPortfolio().getOrDefault(c.getName(),0.0);
            double valUSD = qty*c.getPriceUSD();
            double rate = currencyRates.get(currency);
            total += valUSD*rate;
        }
        lblPortfolio.setText(String.format("Portfolio Value: %.2f %s",total,currency));
    }

    // ---------------- ADMIN PANEL ----------------
    private void showAdminPanel(){
        JFrame admin = new JFrame("Admin Panel");
        admin.setSize(500,400);
        admin.setLocationRelativeTo(null);
        admin.getContentPane().setBackground(BG_DARK);
        admin.setLayout(new GridLayout(4,1,10,10));
        admin.setFont(new Font("Segoe UI", Font.PLAIN, 13));

        JButton btnView = new JButton("View All Cryptos");
        JButton btnAdd = new JButton("Add New Crypto");
        JButton btnRemove = new JButton("Remove Crypto");
        JButton btnLogout = new JButton("Logout (Back to Login)");

        styleFlatButton(btnView, BG_PANEL, Color.WHITE);
        styleFlatButton(btnAdd, ACCENT_SOFT, Color.WHITE);
        styleFlatButton(btnRemove, new Color(150,60,60), Color.WHITE);
        styleFlatButton(btnLogout, new Color(60,60,80), Color.WHITE);

        admin.add(btnView); admin.add(btnAdd); admin.add(btnRemove); admin.add(btnLogout);

        btnView.addActionListener(e -> {
            StringBuilder sb = new StringBuilder("All Cryptos:\n\n");
            for(Crypto c: cryptos){
                sb.append(String.format("%s (%s): $%.2f\n",c.getName(),c.getSymbol(),c.getPriceUSD()));
            }
            JTextArea ta = new JTextArea(sb.toString());
            ta.setEditable(false);
            ta.setBackground(BG_PANEL);
            ta.setForeground(Color.WHITE);
            ta.setFont(new Font("Consolas", Font.PLAIN, 13));
            JOptionPane.showMessageDialog(admin,new JScrollPane(ta),"Cryptos",JOptionPane.INFORMATION_MESSAGE);
        });

        btnAdd.addActionListener(e -> {
            String name = JOptionPane.showInputDialog(admin,"Enter crypto name:");
            if(name == null || name.trim().isEmpty()) return;
            String symbol = JOptionPane.showInputDialog(admin,"Enter symbol:");
            if(symbol == null || symbol.trim().isEmpty()) return;
            String priceStr = JOptionPane.showInputDialog(admin,"Enter price in USD:");
            try{
                double price = Double.parseDouble(priceStr);
                cryptos.add(new Crypto(name,symbol,price));
                JOptionPane.showMessageDialog(admin,"Crypto added!");
            }catch(Exception ex){ JOptionPane.showMessageDialog(admin,"Invalid input!");}
        });

        btnRemove.addActionListener(e -> {
            if(cryptos.isEmpty()){
                JOptionPane.showMessageDialog(admin,"No cryptos to remove.");
                return;
            }
            String[] names = cryptos.stream().map(Crypto::getName).toArray(String[]::new);
            String sel = (String)JOptionPane.showInputDialog(admin,"Select crypto to remove:","Remove",
                    JOptionPane.QUESTION_MESSAGE,null,names,names[0]);
            if(sel!=null){
                cryptos.removeIf(c -> c.getName().equals(sel));
                JOptionPane.showMessageDialog(admin,"Removed "+sel);
            }
        });

        btnLogout.addActionListener(e -> {
            admin.dispose();
            showLogin();
        });

        admin.setVisible(true);
    }

    public static void main(String[] args){
        SwingUtilities.invokeLater(() -> new MainGui());
    }
}
