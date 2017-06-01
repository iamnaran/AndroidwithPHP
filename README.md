# AndroidwithPHP


#DBCONNECT

**

<?php 

        echo "s";
        $host = "localhost";
        $db_user = "root";
        $db_password = "";
        $db_name = "Login";

        $con = mysqli_connect($host, $db_user, $db_password, $db_name);
        if ($con) {
          echo "Connection sucess";
        }else{
          echo "Connection error";
        }

 ?>
 
 **
 
 #DBOPERATIONS
 
 <?php 

	class DbOperations{

		private $con; 

		function __construct(){

			require_once ('DbConnect.php');

			$db = new DbConnect();

			$this->con = $db->connect();

		}

		public function createUser($username, $pass, $email){
			if($this->isUserExist($username,$email)){
				return 0; 
			}else{
				$password = md5($pass);
				$stmt = $this->con->prepare("INSERT INTO `users` (`id`, `username`, `password`, `email`) VALUES (NULL, ?, ?, ?);");
				$stmt->bind_param("sss",$username,$password,$email);

				if($stmt->execute()){
					return 1; 
				}else{
					return 2; 
				}
			}
		}

		public function userLogin($username, $pass){
			$password = md5($pass);
			$stmt = $this->con->prepare("SELECT id FROM users WHERE username = ? AND password = ?");
			$stmt->bind_param("ss",$username,$password);
			$stmt->execute();
			$stmt->store_result(); 
			return $stmt->num_rows > 0; 
		}

		public function getUserByUsername($username){
			$stmt = $this->con->prepare("SELECT * FROM users WHERE username = ?");
			$stmt->bind_param("s",$username);
			$stmt->execute();
			return $stmt->get_result()->fetch_assoc();
		}
		

		private function isUserExist($username, $email){
			$stmt = $this->con->prepare("SELECT id FROM users WHERE username = ? OR email = ?");
			$stmt->bind_param("ss", $username, $email);
			$stmt->execute(); 
			$stmt->store_result(); 
			return $stmt->num_rows > 0; 
		}

	}



#REGISTERUSER

<?php 

require_once '../includes/DbOperations.php';

$response = array(); 

if($_SERVER['REQUEST_METHOD']=='POST'){
	if(
		isset($_POST['username']) and 
			isset($_POST['email']) and 
				isset($_POST['password']))
		{
		//operate the data further 

		$db = new DbOperations(); 

		$result = $db->createUser( 	$_POST['username'],
									$_POST['password'],
									$_POST['email']
								);
		if($result == 1){
			$response['error'] = false; 
			$response['message'] = "User registered successfully";
		}elseif($result == 2){
			$response['error'] = true; 
			$response['message'] = "Some error occurred please try again";			
		}elseif($result == 0){
			$response['error'] = true; 
			$response['message'] = "It seems you are already registered, please choose a different email and username";						
		}

	}else{
		$response['error'] = true; 
		$response['message'] = "Required fields are missing";
	}
}else{
	$response['error'] = true; 
	$response['message'] = "Invalid Request";
}

echo json_encode($response);



#USERLOGIN

<?php 

require_once '../includes/DbOperations.php';

$response = array(); 

if($_SERVER['REQUEST_METHOD']=='POST'){
	if(isset($_POST['username']) and isset($_POST['password'])){
		$db = new DbOperations(); 

		if($db->userLogin($_POST['username'], $_POST['password'])){
			$user = $db->getUserByUsername($_POST['username']);
			$response['error'] = false; 
			$response['id'] = $user['id'];
			$response['email'] = $user['email'];
			$response['username'] = $user['username'];
		}else{
			$response['error'] = true; 
			$response['message'] = "Invalid username or password";			
		}

	}else{
		$response['error'] = true; 
		$response['message'] = "Required fields are missing";
	}
}

echo json_encode($response);

