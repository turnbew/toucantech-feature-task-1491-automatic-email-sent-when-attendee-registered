TASK DATE: 25.04.2018 - FINISHED: 

TASK LEVEL: EASY

TASK SHORT DESCRIPTION: 1491 [
								"Automatic notification sent to event authors and admins (via the contact_email) when 
								an attendee registers to an event.
								Event registration
								""{{name}} has registered to {{event_name}}.
								{{name}}*clickable link to users profile* has registered to {{event_name}}*clickable link to event*.
								Click here*clickable link to .../admin-portal/content/events/attendees/...* to view the event 
								attendee list."
							]

GITHUB REPOSITORY CODE: feature/task-1491-automatic-email-sent-when-attendee-registered


ORIGINAL WORK: https://github.com/BusinessBecause/network-site/tree/feature/task-1491-automatic-email-sent-when-attendee-registered


TEST URL: http://school.networkbecause.local/bbevents/toggle_registration/107


CHANGES
 
	IN FILES: 
	
		\network-site\addons\default\modules\bbevents\controllers\bbevents.php
	
			ADDED CODE

				Inside function register_reservation
				
					//Sending automatic notification about registration to the event's author and site admins in 3 small steps
					#1.: load some necessary models
					$event = $this->bbevent_m->get($event_id);
					
					#2.: getting recipients
					$recipients = array();		
					$recipients[] = $event->email;					//email of the author of the Event
					$recipients[] = Settings::get('contact_email'); //Contact e-mail
							
					#3.: sending e-mails to recipients
					foreach($recipients as $key => $email) 
					{
						$emailsData = array(
							'slug' 				=> 'event_registration_notification',
							'subject'			=>  'New registration to event: ' . $event->title,
							/*Test*/ 'to' 		=> 'lajos.d.05@gmail.com', 
							//'to' 				=> $email,
							'name_link' 		=> '<a href="' . site_url('profile/' . $this->current_user->username) . '" target="_blank">' . $this->current_user->display_name . '</a>',
							'event_link'		=> '<a href="' . site_url('event/' . $event->url_slug) . '" target="_blank">' . $event->title . '</a>',
							'attendees_link'	=> '<a href="' . site_url('admin-portal/content/events/attendees/' . $event_id) . '" target="_blank">here</a>',
							'site_name' 		=> Settings::get('site_name'),
						);
						
						Events::trigger('email', $emailsData, 'array');
					}	
				
	
		
		\network-site\addons\default\modules\bbusers\models\bbuser_m.php
		
			CHANGE function get_all_network_admins
			
				FROM: 
				
					/**
					 * get_all_network_admins
					 * Get all network admins
					 */
					public function get_all_network_admins() {
						$this->db->distinct()
							->select('p.*, u.*')
							->from($this->_table.' u')
							->join('profiles p', 'p.user_id = u.id', 'left')
							->where('u.group_id', 3);

						$res = $this->db->get()->result();
						return $res;
					}
	
				TO: 
				
					    /**
						 * get_all_network_admins
						 * Get all network admins
						 * input
						 *		- $types: string/array : admin types - can be: 1 (toucantech), 2 (payment), 3 (full), 4(db), 0 (all). Default is 3 - full admin. If array then i.e. array(1, 2, ..)
						 */
						public function get_all_network_admins($types = 3) 
						{
							if ( is_array($types) ) {
								$this->db->where_in('group_id', array(1, 2, 3, 4));	
							}
							else {
								switch ($types) {
									case 0	 : $this->db->where_in('group_id', array(1, 2, 3, 4)); break;
									case 1	 : $this->db->where('group_id', 1); break;
									case 2	 : $this->db->where('group_id', 2); break;
									case 3	 : $this->db->where('group_id', 3); break;
									case 4	 : $this->db->where('group_id', 4); break;
								}
							}
							
							$this->db->distinct()
								->select('p.*, u.*')
								->from($this->_table.' u')
								->join('profiles p', 'p.user_id = u.id', 'left');

							$res = $this->db->get()->result();
							return $res;
						}
