# Airline_mysql_python_project
project airline built with python and mysql
import mysql.connector as mysql

host = "localhost"
user = "root"
password = "root"

try:
	db= mysql.connect(host=host, user=user, password=password)
	print("Connected successfully")
except Exception as e:
	print(e)
	print("Failed to Connect")

#creating database
try:
	command_handler = db.cursor()
	command_handler.execute("CREATE DATABASE airline")
	print("airline database created")
except Exception as e:
	print("Could not create database")
	print(e)

#connecting to database
try:
	db1 = mysql.connect(host=host, user=user, password=password, database="airline")
	print("Connected to airline database")
except Exception as e:
	print("Could not connect to database")
	print(e)

#creating tables
try:
	command_handler = db1.cursor()
	command_handler.execute("CREATE TABLE Airport (code VARCHAR(3) PRIMARY KEY, city VARCHAR(64))")
	print("Table Airport created successfully")
except Exception as e:
	print("Could not create table")
	print(e)
try:
	command_handler.execute("CREATE TABLE Flight (flight_id INT PRIMARY KEY, code VARCHAR(3), FOREIGN KEY(code) REFERENCES Airport(code) ON DELETE CASCADE, destination VARCHAR(64), duration INT)")
	print("Table Flight created")
except Exception as e:
	print("Could not create table")
	print(e)
try:
	command_handler.execute("CREATE TABLE Passenger (id INT PRIMARY KEY AUTO_INCREMENT, first VARCHAR(64), last VARCHAR(64), flight_id INT, FOREIGN KEY(flight_id) REFERENCES Flight(flight_id) ON DELETE CASCADE)")
	print("Table Passenger created")
except Exception as e:
	print("Could not create table")
	print(e)

#Keep on displaying options until user wants to exit
while True:
	print("Press 1 to add Airport \nPress 2 to add Flight \nPress 3 to add Passengers \nPress 4 to view details \nPress 5 to exit")
	option = int(input("Option: "))
	#insert into Airport table
	if option == 1:
		code = input("Enter Airport code: ")
		city = input("Enter Airport city: ")
		try:
			query = """INSERT INTO Airport(code, city) VALUES(%s, %s)"""
			query_vals = (code,city)
			command_handler.execute(query,query_vals)
			db1.commit()
		except Exception as e:
			print("Could not add Airport")
			print(e)
	#insert into Flight table
	elif option == 2:
		flight_id = int(input("Enter Flight Id: "))
		code = input("Enter Airport code: ")
		destination = input("Enter Flight destination: ")
		duration = int(input("Enter Flight duration in mins: "))
		try:
			query = """INSERT INTO Flight(flight_id, code, destination, duration) VALUES(%s, %s, %s, %s)"""
			query_vals = (flight_id, code, destination, duration)
			command_handler.execute(query,query_vals)
			db1.commit()
		except Exception as e:
			print("Could not add Flight")
			print(e)
	#inser into passenger table
	elif option == 3:
		flight_id = int(input("Enter Flight Id of Passenger: "))
		first = input("Enter first name: ")
		last = input("Enter last name: ")
		try:
			query = """INSERT INTO Passenger(first, last, flight_id) VALUES(%s,%s,%s)"""
			query_vals = (first, last, flight_id)
			command_handler.execute(query,query_vals)
			db1.commit()
		except Exception as e:
			print("Could not add Passenger")
			print(e)
	elif option == 4:
		print("Press A for Airport details\nPress B for Flight details\nPress C for Passenger details\nPress D to delete a record")
		op = input("Option: ")
		#display airport details
		if op == 'A':
			command_handler.execute("SELECT * FROM Airport")
			tables = command_handler.fetchall()
			for table in tables:
				print(table)
			code = input("Enter Airport code: ")
			query = """SELECT A.code, A.city, F.flight_id, F.destination FROM Airport A INNER JOIN Flight F ON (A.code = F.code) WHERE A.code = (%s)"""
			query_vals = (code,)
			command_handler.execute(query,query_vals)
			tables = command_handler.fetchall()
			if len(tables) == 0:
				print("No Flights for "+ code)
			else:
				for table in tables:
					print(table)
		#display flight details
		elif op == 'B':
			command_handler.execute("SELECT * FROM Flight")
			tables = command_handler.fetchall()
			for table in tables:
				print(table)
			code = input("Enter code: ")
			query = """SELECT A.city, F.flight_id, F.destination, F.duration FROM Airport A INNER JOIN Flight F ON (A.code = F.code) WHERE F.code = (%s)"""
			query_vals = (code,)
			command_handler.execute(query,query_vals)
			tables = command_handler.fetchall()
			if len(tables) == 0:
				print("No Flights for "+ code)
			else:
				for table in tables:
					print(table)
				print('\n')
		#display passenger details
		elif op == "C":
			command_handler.execute("SELECT P.flight_id, F.code, COUNT(P.id)FROM Passenger P INNER JOIN Flight F ON(F.flight_id=P.flight_id) GROUP BY(P.flight_id)")
			tables = command_handler.fetchall()
			for table in tables:
				print(table)
			flight_id = int(input("Enter Flight ID: "))
			query = """SELECT P.id, P.first, P.last, F.code FROM Passenger P INNER JOIN Flight F ON(F.flight_id = P.flight_id) WHERE P.flight_id = %s"""
			query_vals = (flight_id,)
			command_handler.execute(query,query_vals)
			tables = command_handler.fetchall()
			if len(tables) == 0:
				print("No Passenger for Flight ID "+ str(flight_id))
			else:
				for table in tables:
					print(table)
				print('\n')
		elif op == "D":
			print("Press 'a' to delete an Airport\nPress 'b' to delete a Flight\nPress 'c' to delete a Passenger")
			pressed = input("Option: ")
			#delete airport and associtaed flights
			if pressed == 'a':
				code = input("Enter Airport code: ")
				print("NOTE: Deleting "+code+" will also delete associated flights and passengers")
				cont = input("Do you want to continue?y/n ")
				if cont == 'y':
					query="""DELETE FROM Airport WHERE code = %s"""
					query_vals=(code,)
					command_handler.execute(query,query_vals)
					db1.commit()
					print("Deleted successfully")
				else:
					print("Process Terminated")
			#delete flight and associated passengers
			elif pressed == 'b':
				flight_id = int(input("Enter Flight Id: "))
				print("NOTE: Deleting "+str(flight_id)+" will also delete associated passengers")
				cont = input("Do you want to continue?y/n ")
				if cont == 'y':
					query="""DELETE FROM Flight WHERE flight_id = %s"""
					query_vals=(flight_id,)
					command_handler.execute(query,query_vals)
					db1.commit()
					print("Deleted successfully")
				else:
					print("Process Terminated")
			#delete passenger
			elif pressed == 'c':
				passenger_id = int(input("Enter Passenger Id: "))
				cont = input("Do you want to continue?y/n ")
				if cont == 'y':
					query="""DELETE FROM Passenger WHERE id = %s"""
					query_vals=(passenger_id,)
					command_handler.execute(query,query_vals)
					db1.commit()
					print("Deleted successfully")
				else:
					print("Process Terminated")			
			else:
				print("Wrong option")
		else:
			print("Wrong option")
	#exit from while loop
	elif option == 5:
		break
	else:
		print("Incorrect Option")
