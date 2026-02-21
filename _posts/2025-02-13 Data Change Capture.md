---
layout: post
title: "Data Change Capture"
date: 2025-02-13
categories: [Data, Change Capture]
tags: [data, change capture, tutorial]
author: Michael Tien
---

## References

- **[Debezium - Open Source CDC Tool](https://debezium.io/)**
- [Change Data Capture (CDC) Explained](https://www.datadoghq.com/blog/change-data-capture/)
- [Change Data Capture: What it is and How it Works](https://www.striim.com/change-data-capture/)
- [Change Data Capture with Kafka](https://www.confluent.io/blog/change-data-capture-patterns-with-kafka/)

- Microsoft CDC 
    - [Microsoft Learn - Change Data Capture (CDC)](https://learn.microsoft.com/en-us/azure/architecture/data-guide/relational-data/change-data-capture)
    - [Introduction to Change Data Capture](https://docs.microsoft.com/en-us/sql/relational-databases/track-changes/about-change-data-capture-sql-server)

## CDC for MS Access

Microsoft Access does not natively support Change Data Capture (CDC). However, you can implement a custom solution using the following approaches:

1. **Manual Triggers and Logs**: Create triggers in your Access database to log changes manually. This involves writing VBA code to capture insert, update, and delete operations.

2. **Linked Tables with SQL Server**: Use SQL Server as an intermediary. Link your Access tables to SQL Server and enable CDC on the SQL Server tables. This way, changes in Access are captured by SQL Server's CDC.

3. **Third-Party Tools**: Utilize third-party tools that provide CDC capabilities for Access databases. These tools can monitor changes and log them accordingly.

4. **Scheduled Data Export**: Regularly export data from Access to another system that supports CDC, such as SQL Server or a data warehouse, and perform CDC on that system.

Each approach has its own trade-offs in terms of complexity, performance, and maintenance. Choose the one that best fits your requirements and environment.

## Code Example

```csharp
using System;
using System.Data;
using System.Data.OleDb;
using System.Data.SqlClient;
using System.Timers;

class Program
{
    private static Timer _timer;
    private const string AccessConnectionString = "Provider=Microsoft.Jet.OLEDB.4.0;Data Source=RCGForex.mdb;";
    private const string SqlConnectionString = "Server=your_sql_server;Database=your_sql_db;User Id=your_username;Password=your_password;";
    private const string LastModifiedColumn = "LastModified";
    private const int TimerInterval = 60000; // 1 minute

    static void Main()
    {
        _timer = new Timer(TimerInterval);
        _timer.Elapsed += OnTimedEvent;
        _timer.AutoReset = true;
        _timer.Enabled = true;

        Console.WriteLine("Press Enter to exit the program...");
        Console.ReadLine();
    }

    private static void OnTimedEvent(Object source, ElapsedEventArgs e)
    {
        ExportData();
    }

    private static void ExportData()
    {
        using (OleDbConnection accessConnection = new OleDbConnection(AccessConnectionString))
        {
            accessConnection.Open();
            DataTable tables = accessConnection.GetSchema("Tables");

            foreach (DataRow row in tables.Rows)
            {
                string tableName = row["TABLE_NAME"].ToString();
                string query = $"SELECT * FROM [{tableName}] WHERE {LastModifiedColumn} > ?";
                using (OleDbCommand command = new OleDbCommand(query, accessConnection))
                {
                    command.Parameters.AddWithValue("?", DateTime.Now.AddMinutes(-1)); // Assuming LastModified is a DateTime column

                    using (OleDbDataReader reader = command.ExecuteReader())
                    {
                        using (SqlConnection sqlConnection = new SqlConnection(SqlConnectionString))
                        {
                            sqlConnection.Open();
                            while (reader.Read())
                            {
                                string insertQuery = $"INSERT INTO {tableName} (Column1, Column2, {LastModifiedColumn}) VALUES (@Column1, @Column2, @{LastModifiedColumn})";
                                using (SqlCommand sqlCommand = new SqlCommand(insertQuery, sqlConnection))
                                {
                                    sqlCommand.Parameters.AddWithValue("@Column1", reader["Column1"]);
                                    sqlCommand.Parameters.AddWithValue("@Column2", reader["Column2"]);
                                    sqlCommand.Parameters.AddWithValue($"@{LastModifiedColumn}", reader[LastModifiedColumn]);
                                    sqlCommand.ExecuteNonQuery();
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
