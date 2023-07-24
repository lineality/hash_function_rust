# hash_function_rust

```
use std::time::{SystemTime, UNIX_EPOCH};

/*
    designed to be a good-enough hash, not relying on libraries
    that includes string and timestamp

    the timestamp kind of functions as a 'secret key'
    as in crypographic 'signing' and verification
    again: not meant to be super world class,
    but light weight and easy to debug,
    not likely to have unexpected or incomprehensible issues
    no external libraries, trust issues, etc. 
    
    the first few digits are less random, so -> removed
    this also keeps the hash from getting huge so quickly

    to make the numbers more significantly different if even one
    input character is changed: add an additional hash if the 
    current hash is odd/even (even picked here)

    Recommended:
    for timestamp: use this to get sub-second depth in a string

    from datetime import datetime
    # get time
    timestamp_raw = datetime.utcnow()
    # make readable string
    timestamp = timestamp_raw.strftime('%Y%m%d%H%M%S%f')
*/
fn make_hash(input_string: &str, timestamp_string: &str) -> u128 {
    let mut string_to_hash = String::from(input_string);
    string_to_hash.push_str(timestamp_string);

    let mut hash: u128 = 1;
    for this_character in string_to_hash.chars() {
        // println!("this_character {}", this_character);
        // println!("this_character as u128 {}", this_character as u128);

        // Calculate the new hash value using integer arithmetic
        hash = 101 * (hash + this_character as u128);

        // println!("step 1 hash {}", hash);

        // reflip if the hash is even
        if hash % 2 == 0 {
            hash = 101 * (hash + this_character as u128);
        }
        println!("step 2 flip hash {}", hash);

        // Reduce the hash to a 6-digit number by parsing it as a string
        let hash_str = hash.to_string();
        if hash_str.len() > 6 {
            hash = match hash_str[2..].parse() {
                Ok(parsed_hash) => parsed_hash,
                Err(_) => {
                    eprintln!("Failed to parse hash: {}", hash_str);
                    0 // Set a default value or take appropriate action on parsing failure
                }
            };
        }
        if hash_str.len() > 15 {
            hash = match hash_str[4..].parse() {
                Ok(parsed_hash) => parsed_hash,
                Err(_) => {
                    eprintln!("Failed to parse hash: {}", hash_str);
                    0 // Set a default value or take appropriate action on parsing failure
                }
            };
        }
        
        
        
    }
    println!("finished hash {}", hash);
    
    hash
}

fn main() {
    // get current timestamp
    let now = SystemTime::now();
    let timestamp = now.duration_since(UNIX_EPOCH)
                       .expect("Time went backwards")
                       .as_millis()
                       .to_string();

    println!("{}", make_hash("123:123:243:234", &timestamp));

    // compare small change: just the last digit of the time
    println!("{}", make_hash("123:123:243:234", "20221008133518385205"));
    println!("{}", make_hash("123:123:243:234", "20221008133518385206"));

    // small number input: edge case check
    println!("{}", make_hash("1", "3"));
    println!("{}", make_hash("2", "2"));
}

```
