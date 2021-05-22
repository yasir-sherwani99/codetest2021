Refactored Code

My Thoughts
The code seems to be OK. Repository design pattern is used, but this is not fitting design pattern for Laravel. Repositories is sometimes used for Laravel applications, but rather an outlier, than a common sight. Moving the logic into Models and Service classes is better choice. Repositories are not the right place for putting business logic.
  
The following function from app/Repository/BookingRepository.php are refactored.

Original Code
public function getUsersJobs($user_id)
{
	$cuser = User::find($user_id);
        $usertype = '';
        $emergencyJobs = array();
        $noramlJobs = array();
        if ($cuser && $cuser->is('customer')) {
            $jobs = $cuser->jobs()->with('user.userMeta', 'user.average', 'translatorJobRel.user.average', 'language', 'feedback')->whereIn('status', ['pending', 'assigned', 'started'])->orderBy('due', 'asc')->get();
            $usertype = 'customer';
        } elseif ($cuser && $cuser->is('translator')) {
            $jobs = Job::getTranslatorJobs($cuser->id, 'new');
            $jobs = $jobs->pluck('jobs')->all();
            $usertype = 'translator';
        }
        if ($jobs) {
            foreach ($jobs as $jobitem) {
                if ($jobitem->immediate == 'yes') {
                    $emergencyJobs[] = $jobitem;
                } else {
                    $noramlJobs[] = $jobitem;
                }
            }
            $noramlJobs = collect($noramlJobs)->each(function ($item, $key) use ($user_id) {
                $item['usercheck'] = Job::checkParticularJob($user_id, $item);
            })->sortBy('due')->all();
        }

        return ['emergencyJobs' => $emergencyJobs, 'noramlJobs' => $noramlJobs, 'cuser' => $cuser, 'usertype' => $usertype];
}

----------------------------------------------------------------------------------------------------------

Refactored Code
public function getUsersJobs($user_id)
{
     	$cuser = User::find($user_id);
	if(!isset($cuser) || empty($cuser)) {
		abort(404);
	}
		
	$emergencyJobs = [];
	$noramlJobs = [];
					
	$usertype = $this->getUserType($cuser);
	$jobs = $this->getJobs($cuser);
		
	if($jobs) {
		foreach ($jobs as $jobitem) 
		{
                	if ($jobitem->immediate == 'yes') {
                    		$emergencyJobs[] = $jobitem;
                	} else {
                    		$noramlJobs[] = $jobitem;
                	}
            	}
			
		if(!empty($noramlJobs)) {
			$newNormalJobs = $this->getNormalJobsCollection($noramlJobs, $user_id);
		}
	}
		
	return ['emergencyJobs' => $emergencyJobs, 'noramlJobs' => $newNoramlJobs, 'cuser' => $cuser, 'usertype' => $usertype];
				
}

public function getUserType($cuser)
{
	if ($cuser->is('customer')) 
	{							
            $usertype = 'customer';
	} elseif ($cuser->is('translator')) {
	    $usertype = 'translator';
	} else {
	    $usertype = "";
	}
		
	return $usertype;
}
	
public function getJobs($cuser)
{
	if ($cuser->is('customer')) 
	{
            $jobs = $cuser->jobs()
			->with([
				'user.userMeta', 
				'user.average', 
				'translatorJobRel.user.average', 
				'language', 
				'feedback'
			])
			->whereIn('status', ['pending', 'assigned', 'started'])
			->orderBy('due', 'asc')
			->get();
							
	} elseif($cuser->is('translator')) {
			
		$jobs = Job::getTranslatorJobs($cuser->id, 'new');
            	$jobs = $jobs->pluck('jobs')->all();
			
	} else {
			
		$jobs = NULL;
	}
		
	return $jobs;
}

public function getNormalJobsCollection($noramlJobs)
{
	$noramlJobs = collect($noramlJobs)->each(function ($item, $key) use ($user_id) {
				$item['usercheck'] = Job::checkParticularJob($user_id, $item);
			})->sortBy('due')->all();
		
	return $normalJobs;
}

----------------------------------------------------------------------------------------------------
Thanks,
Yasir

