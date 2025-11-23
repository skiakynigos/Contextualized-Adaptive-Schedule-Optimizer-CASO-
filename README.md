import datetime
import random
import time 
class ScheduleError(Exception):
    """Custom exception for scheduling errors (e.g., impossible dependency)."""
    pass

PEAK_FOCUS_HOURS = (9, 13) 
FOCUS_WEIGHTS = {
    "HIGH": 3,
    "MEDIUM": 2,
    "LOW": 1
}
ENERGY_COST = {
    "HIGH": 5,  
    "MEDIUM": 3,
    "LOW": 1
}
MAX_ENERGY = 50 
SIMULATED_WEATHER = {
    (datetime.date.today() + datetime.timedelta(days=0)).strftime("%Y-%m-%d"): "Clear",
    (datetime.date.today() + datetime.timedelta(days=1)).strftime("%Y-%m-%d"): "Rain",
    (datetime.date.today() + datetime.timedelta(days=2)).strftime("%Y-%m-%d"): "Windy",
}
class ScheduleOptimizer:
    def __init__(self, peak_focus_start, peak_focus_end):
        self.peak_focus_start = peak_focus_start
        self.peak_focus_end = peak_focus_end
        self.tasks = {} 
        self.categories = ["work", "personal", "travel", "fitness", "outdoor", "break"]
        self.difficulty_levels = ["HIGH", "MEDIUM", "LOW"]
        self.available_resources = ["Laptop", "Specific Tool", "Phone", "Car"]
    def add_task(self, date_str, name, duration_mins, difficulty, category, 
                 fixed_start=None, depends_on=None, is_outdoor=False, required_resource=None, 
                 group_id=None):
        """Adds a new task with all necessary metadata."""
        if difficulty not in self.difficulty_levels and category != "break":
            raise ScheduleError(f"Invalid difficulty. Use: {self.difficulty_levels}")

        task = {
            "name": name,
            "duration": duration_mins,
            "difficulty": difficulty,
            "category": category,
            "fixed_start": fixed_start,
            "depends_on": depends_on,
            "is_outdoor": is_outdoor,
            "required_resource": required_resource,
            "group_id": group_id,      
            "scheduled_start": None,
            "scheduled_end": None,
            "is_scheduled": False,
            "energy_cost": ENERGY_COST.get(difficulty, 0),
            "priority_score": 0
        }
        
        if date_str not in self.tasks:
            self.tasks[date_str] = []
        
        self.tasks[date_str].append(task)
    def _get_weighted_score(self, task, hour_of_day, final_energy):
        """Calculates a priority score based on task difficulty, focus time, and current energy."""
        score = FOCUS_WEIGHTS.get(task["difficulty"], 0) 
        is_peak = self.peak_focus_start <= hour_of_day < self.peak_focus_end
        if is_peak:
            score *= (1.8 if task["difficulty"] == "HIGH" else 0.5) 
        elif task["difficulty"] == "HIGH":
             score *= 0.7 
        energy_required = task["energy_cost"] * (task["duration"] / 60)
        if final_energy < energy_required and task["difficulty"] != "LOW":
            score *= 0.2
        elif final_energy > (MAX_ENERGY * 0.7) and task["difficulty"] == "HIGH":
            score *= 1.1

        task["priority_score"] = score
        return -score 

    def _simulate_travel_time(self, task_name):
        """Simulates an API call to get real-time travel data."""
        return random.randint(20, 50) 

    def _get_weather_adjustment(self, date_str, is_outdoor):
        """Checks simulated weather and returns a delay."""
        if not is_outdoor:
            return 0, ""

        weather = SIMULATED_WEATHER.get(date_str, "Clear")
        
        if weather == "Rain":
            return 60, "☔ Rain Delay"
        
        return 0, ""

    def _add_mandatory_breaks(self, date_str, start_dt, end_dt):
        """Inserts non-negotiable breaks that must be slotted first."""
        lunch_start = start_dt.replace(hour=12, minute=30, second=0, microsecond=0)
        if start_dt <= lunch_start < end_dt:
             self.add_task(date_str, "Mandatory Lunch", 45, "LOW", "break", fixed_start=lunch_start.strftime("%H:%M"))
             
        afternoon_break_start = start_dt.replace(hour=15, minute=0, second=0, microsecond=0)
        if start_dt <= afternoon_break_start < end_dt:
             self.add_task(date_str, "Afternoon Recharge Break", 30, "LOW", "break", fixed_start=afternoon_break_start.strftime("%H:%M"))
             
        print("✅ Added Mandatory Breaks/Lunch.")

    def optimize_schedule(self, date_str, start_time_hr=8, end_time_hr=20):
        time_blocks = []
        last_end_time = start_dt_limit
        final_energy = MAX_ENERGY
        for task in fixed_tasks:
            task_duration_mins = self._apply_dynamic_adjustments(task, date_str)
            task_end_dt = fixed_start_dt + datetime.timedelta(minutes=task_duration_mins)

            if task_end_dt > end_dt_limit: break 
            last_end_time = task_end_dt
            final_energy -= task["energy_cost"] * (task_duration_mins / 60) 
        if last_end_time < end_dt_limit:
            block_duration = int((end_dt_limit - last_end_time).total_seconds() / 60)
            time_blocks.append((last_end_time, end_dt_limit, block_duration))
        priority_queue = []
        for task in flexible_tasks:
            neg_score = self._get_weighted_score(task, last_end_time.hour, final_energy) 
            heapq.heappush(priority_queue, (neg_score, task["name"], task)) 

        unscheduled_tasks = []
        
        for i, (block_start, block_end, block_duration) in enumerate(time_blocks):
            current_slot_time = block_start
            
            while block_duration > 15 and priority_queue:
               remaining_block_duration = int((block_end - current_slot_time).total_seconds() / 60)

                if task_duration_mins <= remaining_block_duration:
                    final_energy = self._check_and_adjust_energy(task, final_energy, date_str, current_slot_time)

                    task["scheduled_start"] = current_slot_time.strftime("%H:%M")
                    task["scheduled_end"] = (current_slot_time + datetime.timedelta(minutes=task_duration_mins)).strftime("%H:%M")
                    task["is_scheduled"] = True
                    current_slot_time += datetime.timedelta(minutes=task_duration_mins)
                    final_energy -= task["energy_cost"] * (task_duration_mins / 60)
                
                elif remaining_block_duration >= 30: 
                    final_energy -= task["energy_cost"] * (split_duration / 60)
                
                else:
                    unscheduled_tasks.append(task)
                    
                block_duration = int((block_end - current_slot_time).total_seconds() / 60)
        
        self.tasks[date_str] = [t for t in all_tasks if t["is_scheduled"] or t["fixed_start"]]
        self.display_schedule(date_str, final_energy)	

    def _apply_dynamic_adjustments(self, task, date_str):
        """Applies travel and weather adjustments to a task's duration."""
        adjusted_duration = task["duration"]
        
        if task["category"] == "travel":
            adjusted_duration = self._simulate_travel_time(task["name"])
            
        weather_delay, _ = self._get_weather_adjustment(date_str, task["is_outdoor"])
        if weather_delay > 0:
            adjusted_duration += weather_delay
            
        return adjusted_duration

    def _check_and_adjust_energy(self, task, final_energy, date_str, current_slot_time):
        """Inserts a rest break if energy is critically low for a high-focus task."""
        
        if task["difficulty"] == "HIGH":
            energy_needed = task["energy_cost"] * (task["duration"] / 60)
            
            if final_energy < energy_needed and final_energy < MAX_ENERGY * 0.3:

                rest_duration_mins = 30
                rest_task_name = "MANDATORY REST BREAK (Energy)"
                rest_task = {
                    "name": rest_task_name,
                    "duration": rest_duration_mins,
                    "difficulty": "LOW",
                    "category": "break",
                    "fixed_start": current_slot_time.strftime("%H:%M"),
                    "scheduled_start": current_slot_time.strftime("%H:%M"),
                    "scheduled_end": (current_slot_time + datetime.timedelta(minutes=rest_duration_mins)).strftime("%H:%M"),
                    "is_scheduled": True
                }
                self.tasks[date_str].append(rest_task)
                print(f"  [Energy Alert] Inserted 30 min Rest Break at {current_slot_time.strftime('%H:%M')}. Energy Reset.")
                return MAX_ENERGY 
                
        return final_energy


    def display_schedule(self, date_str, final_energy):
        """Displays the finalized schedule in a clean Time Table format."""

        scheduled = sorted([t for t in self.tasks[date_str] if t["is_scheduled"]], key=lambda t: t["scheduled_start"])
        unscheduled = [t for t in self.tasks[date_str] if not t["is_scheduled"] and t["category"] != "break"]
            
        print(f"\n" + "="*80)
        print(f"| {'CONTEXTUALIZED DAILY TIME TABLE: ' + date_str:<78} |")
        print(f"| {'Final Estimated Energy Level: ' + str(round(final_energy, 1)) + '/' + str(MAX_ENERGY):<78} |")
        print("="*80)
        
        if scheduled:
            print(f"| {'Start':<6} | {'End':<6} | {'Dura':<5} | {'Type':<8} | {'Priority':<8} | {'Task Name':<45} |")
            print("|--------|--------|-------|----------|----------|-----------------------------------------------|")
            for t in scheduled:
                required_res = t.get('required_resource') 
                resource_used = required_res[:5] if isinstance(required_res, str) else '-'

                task_type = "FIXED" if t.get("fixed_start") else "FLEX"
                if t['category'] == 'break': task_type = "BREAK"
                priority_tag = "Carried" if t.get("source_day") else t.get('difficulty', '-')
                
                line = f"| {t['scheduled_start']:<6} | {t['scheduled_end']:<6} | {t['duration']:<5} | {task_type:<8} | {priority_tag:<8} | {t['name']:<45} |"
                print(line)
        else:
            print("| No tasks were successfully scheduled for today.                                  |")

        print("="*80)

        if unscheduled:
            print("\n🚨 UNSCHEDULED TASKS:")
            for t in unscheduled:
                deps = f" (Needs: {t.get('depends_on')})" if t.get('depends_on') else ''
                res = f" (Resource: {t.get('required_resource')})" if t.get('required_resource') else ''
                print(f"  - {t['name']} ({t['difficulty']} Focus, {t['duration']} mins){deps}{res}")

def get_valid_input(prompt, validation_func, default=None):
    """General utility function for controlled input."""
    while True:
        user_input = input(prompt).strip()
        if not user_input and default is not None:
            return default
        try:
            return validation_func(user_input)
        except ValueError as e:
            print(f"❌ Invalid input: {e}")

def validate_time(time_str):
    """Validates HH:MM format."""
    if not time_str: return None
    datetime.datetime.strptime(time_str, "%H:%M")
    return time_str

def get_task_details_from_user(optimizer, date_str):
    """Collects all complex task details from the user."""
    
    print("\n--- ENTERING TASK DETAILS ---")
    
    while True:
        name = get_valid_input("Task Name (or 'done' to finish): ", str).strip()
        if name.lower() == 'done':
            break
        
        duration = get_valid_input("Duration (minutes, e.g., 60): ", int)
        fixed_start = get_valid_input("Fixed Start Time (HH:MM) or leave blank for flexible: ", validate_time, default=None)
        category_prompt = f"Category ({', '.join(optimizer.categories[:-1])}): "
        category = get_valid_input(category_prompt, lambda x: x.lower() if x.lower() in optimizer.categories else (_ for _ in () and [x.lower()]), default="work")        
        difficulty_prompt = f"Difficulty ({', '.join(optimizer.difficulty_levels)}): "
        difficulty = get_valid_input(difficulty_prompt, lambda x: x.upper() if x.upper() in optimizer.difficulty_levels else (_ for _ in () and [x.upper()]), default="MEDIUM")
        
        depends_on = get_valid_input("Dependency (Name of prior task, or blank): ", str, default=None)
        is_outdoor = get_valid_input("Is this an outdoor task? (y/n): ", lambda x: x.lower() == 'y', default=False)
        resource_prompt = f"Required Resource ({', '.join(optimizer.available_resources)} or blank): "
        resource = get_valid_input(resource_prompt, lambda x: x if x in optimizer.available_resources else (_ for _ in () and [x]), default=None)
        group_id = get_valid_input("Grouping ID (e.g., 'ProjectAlpha', or blank): ", str, default=None)
        
        try:
            optimizer.add_task(date_str, name, duration, difficulty, category, 
                               fixed_start=fixed_start, depends_on=depends_on, 
                               is_outdoor=is_outdoor, required_resource=resource, 
                               group_id=group_id)
            print(f"Task '{name}' added successfully.")
        except ScheduleError as e:
            print(f"Error adding task: {e}")

def main_cli():
    """Main CLI function to run the optimizer."""
    
    today_str = datetime.date.today().strftime("%Y-%m-%d")
    
    print("Welcome to the Extreme Contextualized Schedule Optimizer!")
    print(f"User's Peak Focus Hours: {PEAK_FOCUS_HOURS[0]}:00 to {PEAK_FOCUS_HOURS[1]}:00")
    
    optimizer = ScheduleOptimizer(PEAK_FOCUS_HOURS[0], PEAK_FOCUS_HOURS[1])
    
    start_hr = get_valid_input("Schedule Start Hour (e.g., 8 for 08:00): ", int, default=8)
    end_hr = get_valid_input("Schedule End Hour (e.g., 18 for 18:00): ", int, default=18)
    
    get_task_details_from_user(optimizer, today_str)
    
    try:
        optimizer.optimize_schedule(today_str, start_time_hr=start_hr, end_time_hr=end_hr)
    except ScheduleError as e:
        print(f"\nCRITICAL SCHEDULING ERROR: {e}")

if __name__ == "__main__":
    main_cli()
