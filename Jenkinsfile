node {

	currentBuild.result = "SUCCESS"
	echo "initial file.........."
	try
            {
		stage('Checkout') {
		checkout scm
		}
		 
		stage('Test') {
		echo "Running: Test"
	        echo $PWD
		sh '''set +x;
		 foodcritic .
                 rubocop .
		'''
		}
	    }

    catch(e)
        {
		echo e 
		throw e
	}
}
