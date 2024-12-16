// Install-Package System.Data.SQLite

using System;
using System.Data.SQLite;

class Program
{
    static void Main()
    {
        string databasePath = "tesla_rent.db";

        // Create the database file if it doesn't exist
        if (!System.IO.File.Exists(databasePath))
        {
            SQLiteConnection.CreateFile(databasePath);
            Console.WriteLine("Database file created.");
        }

        // Set up connection string
        string connectionString = $"Data Source={databasePath};Version=3;";

        using (SQLiteConnection connection = new SQLiteConnection(connectionString))
        {
            connection.Open();

            // Create the 'Customers' table if it doesn't exist
            string createCustomersTableQuery = @"
                CREATE TABLE IF NOT EXISTS Customers (
                    Id INTEGER PRIMARY KEY AUTOINCREMENT,
                    FullName TEXT NOT NULL,
                    Email TEXT NOT NULL
                );
            ";
            using (SQLiteCommand command = new SQLiteCommand(createCustomersTableQuery, connection))
            {
                command.ExecuteNonQuery();
                Console.WriteLine("Customers table created or already exists.");
            }

            // Customer Registration - Prompt customer to input their details
            Console.WriteLine("Please enter your details to register:");

            Console.Write("Full Name (First and Last Name): ");
            string fullName = Console.ReadLine();

            Console.Write("Email: ");
            string email = Console.ReadLine();

            // Insert customer information into the Customers table
            string insertCustomerQuery = @"
                INSERT INTO Customers (FullName, Email) 
                VALUES (@FullName, @Email);
            ";

            using (SQLiteCommand command = new SQLiteCommand(insertCustomerQuery, connection))
            {
                command.Parameters.AddWithValue("@FullName", fullName);
                command.Parameters.AddWithValue("@Email", email);

                command.ExecuteNonQuery();
                Console.WriteLine("\nCustomer registered successfully.");
            }

            // Create the 'Cars' table if it doesn't exist
            string createCarsTableQuery = @"
                CREATE TABLE IF NOT EXISTS Cars (
                    Id INTEGER PRIMARY KEY AUTOINCREMENT,
                    Model TEXT NOT NULL,
                    [Hourly Rate] INTEGER NOT NULL,
                    [Kilometer Rate] INTEGER NOT NULL
                );
            ";
            using (SQLiteCommand command = new SQLiteCommand(createCarsTableQuery, connection))
            {
                command.ExecuteNonQuery();
                Console.WriteLine("Cars table created or already exists.");
            }

            // Create the 'Rentals' table if it doesn't exist
            string createRentalsTableQuery = @"
                CREATE TABLE IF NOT EXISTS Rentals (
                    Id INTEGER PRIMARY KEY AUTOINCREMENT,
                    CustomerId INTEGER NOT NULL,
                    CarId INTEGER NOT NULL,
                    StartTime TEXT NOT NULL,
                    EndTime TEXT NOT NULL,
                    KilometersDriven INTEGER NOT NULL,
                    TotalPayment REAL NOT NULL,
                    FOREIGN KEY (CustomerId) REFERENCES Customers(Id),
                    FOREIGN KEY (CarId) REFERENCES Cars(Id)
                );
            ";
            using (SQLiteCommand command = new SQLiteCommand(createRentalsTableQuery, connection))
            {
                command.ExecuteNonQuery();
                Console.WriteLine("Rentals table created or already exists.");
            }

            // Insert some example cars (if necessary)
            InsertSampleCars(connection);

            // Rent a car (customer selects a car)
            Console.WriteLine("\nAvailable cars for rent:");
            string selectCarsQuery = "SELECT * FROM Cars;";
            using (SQLiteCommand command = new SQLiteCommand(selectCarsQuery, connection))
            using (SQLiteDataReader reader = command.ExecuteReader())
            {
                while (reader.Read())
                {
                    Console.WriteLine($"{reader["Id"]}: {reader["Model"]} - Hourly Rate: {reader["Hourly Rate"]} EUR/h, Kilometer Rate: {reader["Kilometer Rate"]} EUR/km");
                }
            }

            int selectedCarId;
            while (true)
            {
                Console.Write("\nEnter the Car ID to rent (1, 2, or 3): ");
                selectedCarId = Convert.ToInt32(Console.ReadLine());

                // Validate car ID
                if (selectedCarId == 1 || selectedCarId == 2 || selectedCarId == 3)
                {
                    break;  // Exit the loop if a valid ID is entered
                }
                else
                {
                    Console.WriteLine("Invalid Car ID. Please enter a valid Car ID (1, 2, or 3).");
                }
            }

            // Record the start time of the rental
            string startTime = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss");
            Console.WriteLine($"Rental started at: {startTime}");

            // Simulate the car rental for some time (customer enters this info later)
            Console.Write("\nPress Enter when rental has ended...");
            Console.ReadLine();

            // Record the end time of the rental
            string endTime = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss");
            Console.WriteLine($"Rental ended at: {endTime}");

            int kilometersDriven;
            while (true)
            {
                Console.Write("\nEnter kilometers driven: ");
                kilometersDriven = Convert.ToInt32(Console.ReadLine());

                // Validate kilometers driven
                if (kilometersDriven < 0)
                {
                    Console.WriteLine("Kilometers driven cannot be a negative number. Please enter a valid number.");
                }
                else
                {
                    break;  // Exit the loop if a valid number is entered
                }
            }

            // Retrieve car rates (using the selectedCarId)
            string carRateQuery = "SELECT [Hourly Rate], [Kilometer Rate], Model FROM Cars WHERE Id = @CarId;";
            int hourlyRate = 0, kilometerRate = 0;
            string carModel = string.Empty;
            using (SQLiteCommand command = new SQLiteCommand(carRateQuery, connection))
            {
                command.Parameters.AddWithValue("@CarId", selectedCarId); // Use selectedCarId instead of carId
                using (SQLiteDataReader reader = command.ExecuteReader())
                {
                    if (reader.Read())
                    {
                        hourlyRate = Convert.ToInt32(reader["Hourly Rate"]);
                        kilometerRate = Convert.ToInt32(reader["Kilometer Rate"]);
                        carModel = reader["Model"].ToString();
                    }
                }
            }

            // Calculate the payment
            DateTime start = DateTime.Parse(startTime);
            DateTime end = DateTime.Parse(endTime);
            double rentalDurationInHours = (end - start).TotalHours;
            double totalPayment = (rentalDurationInHours * hourlyRate) + (kilometersDriven * kilometerRate);

            // Insert rental data into the Rentals table
            string insertRentalQuery = @"
                INSERT INTO Rentals (CustomerId, CarId, StartTime, EndTime, KilometersDriven, TotalPayment) 
                VALUES (@CustomerId, @CarId, @StartTime, @EndTime, @KilometersDriven, @TotalPayment);
            ";

            using (SQLiteCommand command = new SQLiteCommand(insertRentalQuery, connection))
            {
                command.Parameters.AddWithValue("@CustomerId", GetLastCustomerId(connection));  // Use the ID of the last registered customer
                command.Parameters.AddWithValue("@CarId", selectedCarId);  // Use selectedCarId instead of carId
                command.Parameters.AddWithValue("@StartTime", startTime);
                command.Parameters.AddWithValue("@EndTime", endTime);
                command.Parameters.AddWithValue("@KilometersDriven", kilometersDriven);
                command.Parameters.AddWithValue("@TotalPayment", totalPayment);

                command.ExecuteNonQuery();
                Console.WriteLine("\nRental data inserted successfully.");
                Console.WriteLine($"Total Payment: {totalPayment} EUR.");
            }

            // Optional: Display all rentals
            Console.WriteLine("\nRentals in the database:");
            string selectRentalsQuery = "SELECT * FROM Rentals;";
            using (SQLiteCommand command = new SQLiteCommand(selectRentalsQuery, connection))
            using (SQLiteDataReader reader = command.ExecuteReader())
            {
                while (reader.Read())
                {
                    int customerId = Convert.ToInt32(reader["CustomerId"]);
                    int carId = Convert.ToInt32(reader["CarId"]);

                    // Retrieve the customer name and car model for display
                    string customerNameQuery = "SELECT FullName FROM Customers WHERE Id = @CustomerId;";
                    string customerName = string.Empty;
                    using (SQLiteCommand nameCommand = new SQLiteCommand(customerNameQuery, connection))
                    {
                        nameCommand.Parameters.AddWithValue("@CustomerId", customerId);
                        using (SQLiteDataReader nameReader = nameCommand.ExecuteReader())
                        {
                            if (nameReader.Read())
                            {
                                customerName = nameReader["FullName"].ToString();
                            }
                        }
                    }

                    string carModelQuery = "SELECT Model FROM Cars WHERE Id = @CarId;";
                    string carModelFromDb = string.Empty;
                    using (SQLiteCommand carCommand = new SQLiteCommand(carModelQuery, connection))
                    {
                        carCommand.Parameters.AddWithValue("@CarId", carId);
                        using (SQLiteDataReader carReader = carCommand.ExecuteReader())
                        {
                            if (carReader.Read())
                            {
                                carModelFromDb = carReader["Model"].ToString();
                            }
                        }
                    }

                    // Output rental details with customer and car names
                    Console.WriteLine($"Rental ID: {reader["Id"]}, Customer: {customerName}, Car: {carModelFromDb}, Start Time: {reader["StartTime"]}, End Time: {reader["EndTime"]}, Kilometers: {reader["KilometersDriven"]}, Total Payment: {reader["TotalPayment"]} EUR");
                }
            }
        }
    }

    // Insert sample cars into the Cars table
    static void InsertSampleCars(SQLiteConnection connection)
    {
        string insertCarsQuery = @"
            INSERT OR IGNORE INTO Cars (Model, [Hourly Rate], [Kilometer Rate])
            VALUES
            ('Model 3', 12, 5),
            ('Model Y', 15, 8),
            ('Cybertruck', 10, 7);
        ";
        using (SQLiteCommand command = new SQLiteCommand(insertCarsQuery, connection))
        {
            command.ExecuteNonQuery();
        }
    }

    // Helper function to get the last customer ID
    static int GetLastCustomerId(SQLiteConnection connection)
    {
        string query = "SELECT MAX(Id) FROM Customers;";
        using (SQLiteCommand command = new SQLiteCommand(query, connection))
        {
            return Convert.ToInt32(command.ExecuteScalar());
        }
    }
}
