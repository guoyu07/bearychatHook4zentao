# bearychatHook4zentao
实现将禅道中的动态通知给bearychat

/path_to_zentao/module/action/ext/model/logHistory.php

```php
<?php

/**
     * Log histories for an action.
     * 
     * @param  int    $actionID 
     * @param  array  $changes 
     * @access public
     * @return void
     */
    public function logHistory($actionID, $changes)
    {
        foreach($changes as $change) 
        {
            $change['action'] = $actionID;
            $result = $this->dao->insert(TABLE_HISTORY)->data($change)->exec();
			
			//bearychat notice
			$this->notifyBearychat($actionID, $change['new']);
        }
    }
	
	protected function notifyBearychat($actionID, $text) {
		$action = $this->dao->select('*')->from(TABLE_ACTION)
            ->where('id')->eq($actionID)
            ->fetch();
		
		$_actions = $this->loadModel('action')->transformActions(array($action));
		$action = $_actions[0];
		
		$title = strip_tags(sprintf("%s, %s <em>%s</em> %s %s %s。", $action->date, $action->actor, $action->actionLabel, $action->objectLabel, $action->objectName, 'http://'.(isset($_SERVER['HTTP_X_FORWARDED_HOST']) ? $_SERVER['HTTP_X_FORWARDED_HOST'] : (isset($_SERVER['HTTP_HOST']) ? $_SERVER['HTTP_HOST'] : '')) . $action->objectLink));
		
		return $this->curlPostPayload2Bearychat($title, $text);//, 'project-' . $action->project . ''
	}
	
	protected function curlPostPayload2Bearychat($title, $text, $channel = 'pms') {
		
		$url = 'https://hook.bearychat.com/your_self_params';//简单的定义于此
		
		$str = array('payload' => json_encode(array('text' => $title, 'markdown' => true, 'channel' => $channel, 'attachments' => array(array('text' => strip_tags($text))))));//
		$ch = curl_init();
		curl_setopt($ch, CURLOPT_URL, $url);
		curl_setopt($ch, CURLOPT_POST, 1);
		curl_setopt($ch, CURLOPT_POSTFIELDS, $str);
		curl_setopt($ch, CURLOPT_TIMEOUT, 1);
		
		if (substr($url, 0, 8) == "https://")
		{
			$opt[CURLOPT_SSL_VERIFYHOST] = 1;
			$opt[CURLOPT_SSL_VERIFYPEER] = FALSE;
		}
		curl_setopt_array($ch, $opt);
		curl_exec($ch);
		curl_close($ch);
	}
```
