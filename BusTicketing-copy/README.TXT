import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class BusTicketingSystemGUI {
    private static final String[] DESTINATIONS = {"Balanga", "Abucay", "Samal", "Orani", "Hermosa", "Dinalupihan"};
    private JFrame frame;
    private JComboBox<String> destinationBox;
    private JComboBox<String> startingPointBox;
    private JTextField moneyField;
    private JTextArea outputArea;
    private JButton buyTicketButton;
    private JButton clearButton;
    private JLabel fareLabel;
    private JComboBox<String> discountBox;
    private JTextField quantityField;
    private JButton decrementButton;
    private JButton incrementButton;
    private JButton checkoutButton; // New Checkout button
    private List<Integer> availableSeats;

    public static void main(String[] args) {
        SwingUtilities.invokeLater(BusTicketingSystemGUI::new);
    }

    public BusTicketingSystemGUI() {
        frame = new JFrame("Bus Ticketing System");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 600);
        frame.setLayout(null);

        // Center the frame
        frame.setLocationRelativeTo(null);

        JLabel destinationLabel = new JLabel("Select Destination:");
        destinationLabel.setBounds(20, 20, 150, 25);
        frame.add(destinationLabel);

        destinationBox = new JComboBox<>(DESTINATIONS);
        destinationBox.setBounds(180, 20, 150, 25);
        frame.add(destinationBox);

        JLabel startingPointLabel = new JLabel("Select Starting Point:");
        startingPointLabel.setBounds(20, 60, 150, 25);
        frame.add(startingPointLabel);

        startingPointBox = new JComboBox<>(DESTINATIONS);
        startingPointBox.setBounds(180, 60, 150, 25);
        frame.add(startingPointBox);

        fareLabel = new JLabel("Fare: ");
        fareLabel.setBounds(20, 100, 200, 25);
        frame.add(fareLabel);

        decrementButton = new JButton("-");
        decrementButton.setBounds(180, 100, 45, 25);
        frame.add(decrementButton);

        quantityField = new JTextField("1");
        quantityField.setBounds(240, 100, 30, 25);
        quantityField.setEditable(true);
        frame.add(quantityField);

        incrementButton = new JButton("+");
        incrementButton.setBounds(285, 100, 45, 25);
        frame.add(incrementButton);

        JLabel moneyLabel = new JLabel("Enter your money:");
        moneyLabel.setBounds(20, 140, 150, 25);
        frame.add(moneyLabel);

        moneyField = new JTextField();
        moneyField.setBounds(180, 140, 150, 25);
        frame.add(moneyField);

        JLabel discountLabel = new JLabel("Select Discount:");
        discountLabel.setBounds(20, 180, 150, 25);
        frame.add(discountLabel);

        String[] discounts = {"None", "Student (20%)", "Senior Citizen / PWD (20%)"};
        discountBox = new JComboBox<>(discounts);
        discountBox.setBounds(180, 180, 150, 25);
        frame.add(discountBox);

        buyTicketButton = new JButton("Confirm");
        buyTicketButton.setBounds(20, 220, 150, 25);
        frame.add(buyTicketButton);

        clearButton = new JButton("Clear");
        clearButton.setBounds(180, 220, 150, 25);
        frame.add(clearButton);

        checkoutButton = new JButton("Checkout"); // New Checkout button
        checkoutButton.setBounds(100, 260, 200, 25); // Positioning the Checkout button
        checkoutButton.setEnabled(false); // Initially disabled
        frame.add(checkoutButton);

        outputArea = new JTextArea();
        outputArea.setBounds(10, 300, 360, 199);
        outputArea.setEditable(false);
        frame.add(outputArea);

        destinationBox.addActionListener(new UpdateFareAction());
        startingPointBox.addActionListener(new UpdateFareAction());
        decrementButton.addActionListener(new QuantityAction(-1));
        incrementButton.addActionListener(new QuantityAction(1));

        buyTicketButton.addActionListener(new BuyTicketAction());
        clearButton.addActionListener(new ClearAction());
        checkoutButton.addActionListener(new CheckoutAction()); // Add ActionListener for Checkout button

        frame.setVisible(true);

        // Initialize available seats
        initializeSeats();
    }

    private void initializeSeats() {
        availableSeats = new ArrayList<>();
        for (int i = 1; i <= 50; i++) {
            availableSeats.add(i);
        }
        Collections.sort(availableSeats);
    }

    private class UpdateFareAction implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            updateFare();
        }
    }

    private void updateFare() {
        int destinationIndex = destinationBox.getSelectedIndex();
        int startingPointIndex = startingPointBox.getSelectedIndex();
        int fare = calculateFare(destinationIndex, startingPointIndex);
        fareLabel.setText("Fare: ₱" + fare);
    }

    private class QuantityAction implements ActionListener {
        private int delta;

        public QuantityAction(int delta) {
            this.delta = delta;
        }

        @Override
        public void actionPerformed(ActionEvent e) {
            int quantity = Integer.parseInt(quantityField.getText());
            quantity += delta;
            if (quantity < 1) {
                quantity = 1;
            }
            quantityField.setText(String.valueOf(quantity));
        }
    }

    private class BuyTicketAction implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            try {
                int destinationIndex = destinationBox.getSelectedIndex();
                int startingPointIndex = startingPointBox.getSelectedIndex();
                int fare = calculateFare(destinationIndex, startingPointIndex);

                String discount = (String) discountBox.getSelectedItem();
                if ("Student (20%)".equals(discount) || "Senior Citizen / PWD (20%)".equals(discount)) {
                    fare *= 0.8; // Apply 20% discount
                }

                int quantity = Integer.parseInt(quantityField.getText());
                if (quantity > availableSeats.size()) {
                    outputArea.setText("Not enough seats available.");
                    return;
                }

                int totalFare = fare * quantity;
                int money = Integer.parseInt(moneyField.getText());
                if (money >= totalFare) {
                    int change = money - totalFare;
                    int ticketNumber = generateTicketNumber();
                    String busNo = "BUS-" + (int) (Math.random() * 1000) + 100;

                    // Generate seat numbers
                    List<Integer> assignedSeats = assignSeats(quantity);
                    if (assignedSeats.isEmpty()) {
                        outputArea.setText("Unable to assign adjacent seats. Please try reducing the quantity or try again.");
                        return;
                    }

                    LocalDateTime dateTime = LocalDateTime.now();
                    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");
                    String formattedDateTime = dateTime.format(formatter);

                    // Format and display ticket
                    displayTicket(busNo, assignedSeats, formattedDateTime, ticketNumber, destinationIndex, startingPointIndex, totalFare, change, fare, discount, quantity, money);
                    checkoutButton.setEnabled(true); // Enable Checkout button after displaying ticket
                    buyTicketButton.setEnabled(false); // Disable Buy Ticket button after purchase
                } else {
                    outputArea.setText("You don't have enough money.");
                }
            } catch (NumberFormatException ex) {
                outputArea.setText("Please enter a valid amount of money.");
            }
        }

        private List<Integer> assignSeats(int quantity) {
            List<Integer> assignedSeats = new ArrayList<>();
            for (int i = 0; i <= availableSeats.size() - quantity; i++) {
                boolean consecutive = true;
                for (int j = 0; j < quantity - 1; j++) {
                    if (availableSeats.get(i + j) + 1 != availableSeats.get(i + j + 1)) {
                        consecutive = false;
                        break;
                    }
                }
                if (consecutive) {
                    for (int j = 0; j < quantity; j++) {
                        assignedSeats.add(availableSeats.remove(i));
                    }
                    break;
                }
            }
            return assignedSeats;
        }

        private void displayTicket(String busNo, List<Integer> assignedSeats, String formattedDateTime,
                                   int ticketNumber, int destinationIndex, int startingPointIndex,
                                   int totalFare, int change, int fare, String discount, int quantity, int money) {
            StringBuilder ticket = new StringBuilder();

            ticket.append("                             Receipt : ").append(ticketNumber).append("\n")
                    .append(formattedDateTime).append("\n\n")
                    .append(String.format("                              BUS NO. %-10s\n", busNo))
                    .append(String.format("                              Seats: %s\n", assignedSeats))
                    .append(String.format("                              Route: %-10s to %-10s\n", DESTINATIONS[startingPointIndex], DESTINATIONS[destinationIndex]))
                    .append("                            ------------------------------------------------\n")
                    .append(String.format("                              Fare ................................ ₱%d\n", fare))
                    .append(String.format("                              Discount .......................... %s\n", discount))
                    .append(String.format("                              Quantity ........................... %d\n", quantity))
                    .append(String.format("                              Total Fare ......................... ₱%d\n", totalFare))
                    .append(String.format("                              Cash .............................. ₱%d\n", money))
                    .append("                            ------------------------------------------------\n")
                    .append(String.format("                              Total ............................... ₱%d\n", totalFare))
                    .append(String.format("                              Change .......................... ₱%d\n", change));

            outputArea.setText(ticket.toString());
        }
    }

    private class ClearAction implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            // Reset all input fields and enable the Buy Ticket button
            destinationBox.setSelectedIndex(0);
            startingPointBox.setSelectedIndex(0);
            moneyField.setText("");
            discountBox.setSelectedIndex(0);
            quantityField.setText("1");
            fareLabel.setText("Fare: ");
            outputArea.setText("");
            buyTicketButton.setEnabled(true); // Enable the Buy Ticket button for a new transaction
            checkoutButton.setEnabled(false); // Disable Checkout button

            // Reinitialize seats
            initializeSeats();
        }
    }

    private class CheckoutAction implements ActionListener {
        @Override
        public void actionPerformed(ActionEvent e) {
            // Create a new JFrame for checkout panel
            JFrame checkoutFrame = new JFrame("Ticket Details");
            checkoutFrame.setSize(400, 400);
            checkoutFrame.setLayout(new BorderLayout());
            checkoutFrame.setLocationRelativeTo(null); // Center the frame

            JTextArea ticketDetails = new JTextArea(outputArea.getText());
            ticketDetails.setEditable(false);

            JScrollPane scrollPane = new JScrollPane(ticketDetails);
            checkoutFrame.add(scrollPane, BorderLayout.CENTER);

            // Create a "Done" button
            JButton doneButton = new JButton("Done");
            doneButton.addActionListener(new ActionListener() {
                @Override
                public void actionPerformed(ActionEvent e) {
                    // Reset main panel and enable necessary components
                    clearFieldsAndReset();
                    checkoutFrame.dispose(); // Close the checkout frame
                }
            });

            // Add the "Done" button to the bottom of the panel
            JPanel buttonPanel = new JPanel(new FlowLayout(FlowLayout.CENTER));
            buttonPanel.add(doneButton);
            checkoutFrame.add(buttonPanel, BorderLayout.SOUTH);

            checkoutFrame.setVisible(true);
        }

        private void clearFieldsAndReset() {
            // Reset all input fields and enable the Buy Ticket button
            destinationBox.setSelectedIndex(0);
            startingPointBox.setSelectedIndex(0);
            moneyField.setText("");
            discountBox.setSelectedIndex(0);
            quantityField.setText("1");
            fareLabel.setText("Fare: ");
            outputArea.setText("");
            buyTicketButton.setEnabled(true); // Enable the Buy Ticket button for a new transaction
            checkoutButton.setEnabled(false); // Disable Checkout button

            // Reinitialize seats
            initializeSeats();
        }
    }

    private int calculateFare(int destinationIndex, int startingPointIndex) {
        int baseFare = 30;
        int fareIncrement = 5;
        return baseFare + (Math.abs(destinationIndex - startingPointIndex) * fareIncrement);
    }

    private int generateTicketNumber() {
        return (int) (Math.random() * 9000) + 1000;
    }
}

