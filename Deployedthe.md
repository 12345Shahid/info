Project EduHalal


create a website for students of different levels in Bangladesh which will help them in homework. It will be simple chat response website like ChatGPT. With some different categories like SSC/HSC/admission/university/below SSC. But for simplicity in every case it will use the same AI model which is geminy 2.0. And the same interface, but with the different title for each category. Each category will have a different UI, but as I say, they can have the same design for that with a little differency ,  so the user will have a category selection page. Make sure that the user can also upload images or in short I should say attachments as well for showcasing the homework . It is like normal chatting functionality..  and yes, it should have a beautiful but simple and modern landing page 
[ use next.js as my preferred framework] . 


Below is the Gemini API key by me for the generation of the content 








GOOGLE_API_KEY=AIzaSyDFX8oR3jKgLdKnja6JEPPYTBPwhxw_7r0

NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImtiZ2JiaGNyZ2hrZG93YXR1cG1kIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDQ0NTA1MzcsImV4cCI6MjA2MDAyNjUzN30.fm3vOuo_hBZ9C9jhfjQ1WppDRZjmv6t05FrOThY5_bY
NEXT_PUBLIC_SUPABASE_URL=https://kbgbbhcrghkdowatupmd.supabase.co 


SUPABASE project name edu.1







— one



-- Chat History Table
CREATE TABLE IF NOT EXISTS chat_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT DEFAULT 'anonymous', -- Will be used for auth later
  category TEXT NOT NULL,
  title TEXT NOT NULL DEFAULT 'New Chat',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Chat Messages Table
CREATE TABLE IF NOT EXISTS chat_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  chat_id UUID REFERENCES chat_history(id) ON DELETE CASCADE,
  role TEXT NOT NULL CHECK (role IN ('user', 'assistant')),
  content TEXT NOT NULL,
  has_attachment BOOLEAN DEFAULT false,
  attachment_url TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Notes Table 
CREATE TABLE IF NOT EXISTS notes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT DEFAULT 'anonymous', -- Will be used for auth later
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  folder_id UUID REFERENCES note_folders(id) ON DELETE SET NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  tags TEXT[] DEFAULT ARRAY[]::TEXT[]
);

-- Note Folders Table
CREATE TABLE IF NOT EXISTS note_folders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT DEFAULT 'anonymous', -- Will be used for auth later
  name TEXT NOT NULL,
  parent_id UUID REFERENCES note_folders(id) ON DELETE CASCADE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- User Preferences Table
CREATE TABLE IF NOT EXISTS user_preferences (
  user_id TEXT PRIMARY KEY DEFAULT 'anonymous',
  theme TEXT DEFAULT 'light',
  language TEXT DEFAULT 'en',
  reading_level TEXT DEFAULT 'standard',
  show_step_by_step BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Create indexes for better performance
CREATE INDEX IF NOT EXISTS chat_history_user_id_idx ON chat_history(user_id);
CREATE INDEX IF NOT EXISTS chat_messages_chat_id_idx ON chat_messages(chat_id);
CREATE INDEX IF NOT EXISTS notes_user_id_idx ON notes(user_id);
CREATE INDEX IF NOT EXISTS notes_folder_id_idx ON notes(folder_id);
CREATE INDEX IF NOT EXISTS note_folders_user_id_idx ON note_folders(user_id);
CREATE INDEX IF NOT EXISTS note_folders_parent_id_idx ON note_folders(parent_id);

-- Create function to update the updated_at timestamp
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create triggers to automatically update the updated_at timestamp
CREATE TRIGGER update_chat_history_updated_at
BEFORE UPDATE ON chat_history
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_notes_updated_at
BEFORE UPDATE ON notes
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_note_folders_updated_at
BEFORE UPDATE ON note_folders
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_user_preferences_updated_at
BEFORE UPDATE ON user_preferences
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();

-- Fix the note_folders reference in notes table
ALTER TABLE notes DROP CONSTRAINT IF EXISTS notes_folder_id_fkey;
ALTER TABLE notes ADD CONSTRAINT notes_folder_id_fkey FOREIGN KEY (folder_id) REFERENCES note_folders(id) ON DELETE SET NULL;





— two


-- Note Folders Table (Create this FIRST)
CREATE TABLE IF NOT EXISTS note_folders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT DEFAULT 'anonymous',
  name TEXT NOT NULL,
  parent_id UUID REFERENCES note_folders(id) ON DELETE CASCADE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Chat History Table
CREATE TABLE IF NOT EXISTS chat_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT DEFAULT 'anonymous',
  category TEXT NOT NULL,
  title TEXT NOT NULL DEFAULT 'New Chat',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Chat Messages Table
CREATE TABLE IF NOT EXISTS chat_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  chat_id UUID REFERENCES chat_history(id) ON DELETE CASCADE,
  role TEXT NOT NULL CHECK (role IN ('user', 'assistant')),
  content TEXT NOT NULL,
  has_attachment BOOLEAN DEFAULT false,
  attachment_url TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Notes Table (Create this AFTER note_folders)
CREATE TABLE IF NOT EXISTS notes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id TEXT DEFAULT 'anonymous',
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  folder_id UUID REFERENCES note_folders(id) ON DELETE SET NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  tags TEXT[] DEFAULT ARRAY[]::TEXT[]
);

-- User Preferences Table
CREATE TABLE IF NOT EXISTS user_preferences (
  user_id TEXT PRIMARY KEY DEFAULT 'anonymous',
  theme TEXT DEFAULT 'light',
  language TEXT DEFAULT 'en',
  reading_level TEXT DEFAULT 'standard',
  show_step_by_step BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Create indexes for better performance
CREATE INDEX IF NOT EXISTS chat_history_user_id_idx ON chat_history(user_id);
CREATE INDEX IF NOT EXISTS chat_messages_chat_id_idx ON chat_messages(chat_id);
CREATE INDEX IF NOT EXISTS notes_user_id_idx ON notes(user_id);
CREATE INDEX IF NOT EXISTS notes_folder_id_idx ON notes(folder_id);
CREATE INDEX IF NOT EXISTS note_folders_user_id_idx ON note_folders(user_id);
CREATE INDEX IF NOT EXISTS note_folders_parent_id_idx ON note_folders(parent_id);

-- Create function to update the updated_at timestamp
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create triggers to automatically update the updated_at timestamp
CREATE TRIGGER update_chat_history_updated_at
BEFORE UPDATE ON chat_history
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_notes_updated_at
BEFORE UPDATE ON notes
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_note_folders_updated_at
BEFORE UPDATE ON note_folders
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_user_preferences_updated_at
BEFORE UPDATE ON user_preferences
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();




— three




-- Enable Row Level Security (RLS) on all tables
ALTER TABLE chat_history ENABLE ROW LEVEL SECURITY;
ALTER TABLE chat_messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE notes ENABLE ROW LEVEL SECURITY;
ALTER TABLE note_folders ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_preferences ENABLE ROW LEVEL SECURITY;

-- Update user_id fields to use auth.uid() instead of 'anonymous'
ALTER TABLE chat_history ALTER COLUMN user_id SET DEFAULT auth.uid()::text;
ALTER TABLE notes ALTER COLUMN user_id SET DEFAULT auth.uid()::text;
ALTER TABLE note_folders ALTER COLUMN user_id SET DEFAULT auth.uid()::text;
ALTER TABLE user_preferences ALTER COLUMN user_id SET DEFAULT auth.uid()::text;

-- Create policies for chat_history table
CREATE POLICY "Users can view their own chat history"
  ON chat_history FOR SELECT
  USING (user_id = auth.uid()::text);

CREATE POLICY "Users can insert their own chat history"
  ON chat_history FOR INSERT
  WITH CHECK (user_id = auth.uid()::text);

CREATE POLICY "Users can update their own chat history"
  ON chat_history FOR UPDATE
  USING (user_id = auth.uid()::text);

CREATE POLICY "Users can delete their own chat history"
  ON chat_history FOR DELETE
  USING (user_id = auth.uid()::text);

-- Create policies for chat_messages table (based on chat_id)
CREATE POLICY "Users can view messages in their chats"
  ON chat_messages FOR SELECT
  USING (chat_id IN (SELECT id FROM chat_history WHERE user_id = auth.uid()::text));

CREATE POLICY "Users can insert messages in their chats"
  ON chat_messages FOR INSERT
  WITH CHECK (chat_id IN (SELECT id FROM chat_history WHERE user_id = auth.uid()::text));

CREATE POLICY "Users can update messages in their chats"
  ON chat_messages FOR UPDATE
  USING (chat_id IN (SELECT id FROM chat_history WHERE user_id = auth.uid()::text));

CREATE POLICY "Users can delete messages in their chats"
  ON chat_messages FOR DELETE
  USING (chat_id IN (SELECT id FROM chat_history WHERE user_id = auth.uid()::text));

-- Create policies for notes table
CREATE POLICY "Users can view their own notes"
  ON notes FOR SELECT
  USING (user_id = auth.uid()::text);

CREATE POLICY "Users can insert their own notes"
  ON notes FOR INSERT
  WITH CHECK (user_id = auth.uid()::text);

CREATE POLICY "Users can update their own notes"
  ON notes FOR UPDATE
  USING (user_id = auth.uid()::text);

CREATE POLICY "Users can delete their own notes"
  ON notes FOR DELETE
  USING (user_id = auth.uid()::text);

-- Create policies for note_folders table
CREATE POLICY "Users can view their own folders"
  ON note_folders FOR SELECT
  USING (user_id = auth.uid()::text);

CREATE POLICY "Users can insert their own folders"
  ON note_folders FOR INSERT
  WITH CHECK (user_id = auth.uid()::text);

CREATE POLICY "Users can update their own folders"
  ON note_folders FOR UPDATE
  USING (user_id = auth.uid()::text);

CREATE POLICY "Users can delete their own folders"
  ON note_folders FOR DELETE
  USING (user_id = auth.uid()::text);

-- Create policies for user_preferences table
CREATE POLICY "Users can view their own preferences"
  ON user_preferences FOR SELECT
  USING (user_id = auth.uid()::text);

CREATE POLICY "Users can insert their own preferences"
  ON user_preferences FOR INSERT
  WITH CHECK (user_id = auth.uid()::text);

CREATE POLICY "Users can update their own preferences"
  ON user_preferences FOR UPDATE
  USING (user_id = auth.uid()::text);

-- Migration: Update existing records with anonymous user_id
-- This will associate existing anonymous data with new users during their first login
-- Create a function that can be called from client-side after authentication
CREATE OR REPLACE FUNCTION migrate_anonymous_data(p_user_id TEXT)
RETURNS VOID AS $$
BEGIN
  -- Update chat history
  UPDATE chat_history
  SET user_id = p_user_id
  WHERE user_id = 'anonymous';
  
  -- Update notes
  UPDATE notes
  SET user_id = p_user_id
  WHERE user_id = 'anonymous';
  
  -- Update note folders
  UPDATE note_folders
  SET user_id = p_user_id
  WHERE user_id = 'anonymous';
  
  -- Update user preferences
  UPDATE user_preferences
  SET user_id = p_user_id
  WHERE user_id = 'anonymous';
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
































Project QRW 











The database is not actually actively used here .


Okay, this is the supabase url  https://sarpezfiuycuaptedrit.supabase.co

this is the supabase public key  eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InNhcnBlemZpdXljdWFwdGVkcml0Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDQyMTA0MjIsImV4cCI6MjA1OTc4NjQyMn0.8qXeoFpNgb6xjKi6k61X_QzMFOBjPhK1H7gmsfXysfg 


PhJ#B4uFTxxX4A* = supabase database password also saved in Google passwords



supabase project name quran.1















