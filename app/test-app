import unittest
import socket
from app import app

class FlaskTestCase(unittest.TestCase):

    # Ensure that Flask was set up correctly
    def test_index_STATUS(self):
        tester = app.test_client(self)
        response = tester.get('/', content_type='html/text')
        self.assertEqual(response.status_code, 200)


    def test_index_IP(self):
        host_name = socket.gethostname()
        host_ip = socket.gethostbyname(host_name)
        tester = app.test_client(self)
        response = tester.get('/', content_type='html/text')
        IP = bytes(host_ip, 'utf-8')
        HOSTNAME = bytes(host_name, 'utf-8')
        message = "key is not in container."
        # assertIn() to check if key is in container
        self.assertIn(HOSTNAME,response.data)
        self.assertIn(IP, response.data, message)
        print(host_ip)
        print(host_name)


if __name__ == '__main__':
    unittest.main()
